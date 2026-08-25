# Patrol Integration Tests

**Last Updated**: 2026-08-25  
**Context**: Integration test suite using `patrol_finders` for UI automation on the QA flavor.

---

## Overview

The patrol test suite lives in `syncro-flutter/integration_test/patrol/`. It uses `patrol_finders` (widget-tree only — no native automation) to drive the real app on a connected Simulator or device.

```
integration_test/patrol/
  utils/
    app_test_setup.dart    # Shared setup: startApp, auth helpers, withSocketFilter
    get_qa_token.dart      # One-time utility to extract QA tokens
  features/
    authentication_test.dart
    tickets_test.dart
    assets_test.dart
    settings_test.dart
    end_users_test.dart
    ticket_secondary_test.dart
    ticket_mutations_test.dart
    appointment_mutations_test.dart
    chat_timer_mutations_test.dart
    form_validation_test.dart
```

---

## How to Run

### Minimum viable command

```bash
cd syncro-flutter
fvm flutter test \
  integration_test/patrol/features/<test_file>.dart \
  --flavor qa \
  -d <SIMULATOR_DEVICE_ID> \
  --reporter expanded
```

### With QA authentication (required for most tests)

```bash
fvm flutter test \
  integration_test/patrol/features/<test_file>.dart \
  --flavor qa \
  -d <SIMULATOR_DEVICE_ID> \
  --dart-define=QA_REFRESH_TOKEN=<refresh_token> \
  --dart-define=QA_SUBDOMAIN=<subdomain> \
  --dart-define=PATROL=true \
  --reporter expanded
```

`--dart-define=PATROL=true` is **mandatory** — without it, the iOS native notification
permission dialog blocks every test run requiring manual interaction.
`NotificationManager.requestPermissions()` checks this flag and returns `true` immediately
when running under patrol.

Get `SIMULATOR_DEVICE_ID` with: `xcrun simctl list devices booted`

### Run all tests (sequentially — they share a device)

```bash
for f in integration_test/patrol/features/*_test.dart; do
  fvm flutter test "$f" --flavor qa -d <DEVICE_ID> \
    --dart-define=QA_REFRESH_TOKEN=<token> \
    --dart-define=QA_SUBDOMAIN=<subdomain> \
    --dart-define=PATROL=true \
    --reporter compact
done
```

---

## Android Emulator Setup

Two Android-emulator-specific failures reliably show up on a machine that hasn't run this suite on Android before. Fix both **before** attempting any Android QA login.

### GPU crash during OAuth login (WebView / Cloudflare Turnstile)

**Symptom**: `fvm flutter run --flavor qa -d <emulator>` installs and launches fine, but the app fatal-crashes the instant the OAuth WebView tries to render the Cloudflare Turnstile challenge. Log shows:
```
E/EGL_emulation: eglCreateContext: EGL_BAD_CONFIG: no ES 3.1 support
F/chromium: [FATAL:ui/gl/gl_version_info.cc:73] Chrome runs only on top of OpenGL ES through either ANGLE or native
F/libc: Fatal signal 5 (SIGTRAP) ... libwebviewchromium.so
Lost connection to device.
```
**Root cause**: GPU-passthrough incompatibility between the host GPU and the AVD's default `hw.gpu.mode=auto` — the system WebView needs GLES 3.1 for the Turnstile widget and doesn't get it through host passthrough on some Mac/AVD combos.

**Fix** — launch the emulator with software rendering instead of the default `flutter emulators --launch`:
```bash
~/Library/Android/sdk/emulator/emulator -avd <AVD_NAME> -gpu swiftshader_indirect
```
Confirmed working: the boot log then shows `GPU Version=[OpenGL ES 3.1.0 (ANGLE ...)]` and login renders normally. Tradeoff is slower rendering — acceptable for QA login/test runs. Treat this as the **default** way to launch this project's Android QA emulators, not a fallback-only workaround.

**Relaunch gotcha**: `adb -s <device> emu kill` does not synchronously tear down the process. Launching a new instance of the same AVD a few seconds later throws `FATAL | Running multiple emulators with the same AVD is an experimental feature.` Before relaunching, confirm the old process is fully gone:
```bash
ps aux | grep -i "emulator.*<AVD_NAME>"
pkill -f "emulator.*<AVD_NAME>"   # only if still present
```

### `INSTALL_FAILED_INSUFFICIENT_STORAGE`

**Symptom**: `adb: failed to install ... Failure [INSTALL_FAILED_INSUFFICIENT_STORAGE: Failed to override installation location]` after several install/reinstall cycles in one session.

**Root cause**: `disk.dataPartition.size` in `~/.android/avd/<AVD_NAME>.avd/config.ini` defaults small (6G was observed on a Pixel_9_Pro_XL AVD) and fills up fast from repeated Flutter debug APK installs + dexopt + build caches across a test session. Confirm with `adb -s <device> shell df -h /data`.

**Fix**: bump the partition and cold-boot to apply it:
```bash
# in ~/.android/avd/<AVD_NAME>.avd/config.ini
disk.dataPartition.size=12G
```
```bash
~/Library/Android/sdk/emulator/emulator -avd <AVD_NAME> -gpu swiftshader_indirect -wipe-data
```
`-wipe-data` is **destructive** — it erases everything on the AVD. Only safe to run when there's no session on it worth keeping (check with the human first if unsure). Consider setting a larger default partition size when creating new Android QA AVDs for this project to avoid hitting this mid-session.

### Physical OEM device: app doesn't auto-launch after install

On a real device with a heavily-customized OEM Android skin (confirmed on a Vivo/Funtouch OS Android 15 phone — likely also affects Xiaomi/MIUI, Oppo/ColorOS), after `flutter run`/`flutter test` installs the debug APK, the app does **not** auto-launch to foreground. `adb shell dumpsys activity activities | grep mFocusedApp` still shows the launcher focused, and `adb shell ps -A | grep <package>` shows no process running — even though the app icon appears correctly on the home screen. No permission dialog, no logcat error: it's the OEM's autostart-management silently blocking a freshly-sideloaded app.

**Fix**: the human must manually tap the app icon once from the home screen to open it for the first time. Subsequent `flutter run`/`flutter test` launches work normally afterward (within that install).

---

## QA Authentication

### Why dart-define tokens?

The app uses OAuth2 via WebView for login. `patrolWidgetTest` cannot interact with HTML inputs inside a WebView — there are no Flutter widgets for the credentials form. So tests cannot log in programmatically.

The solution: seed a pre-obtained refresh token into `FlutterSecureStorage` **before** `app.main()` runs. `AuthenticationCubit` reads this token on startup, calls the `me` endpoint, auto-refreshes the access token if stale, and lands on the home screen.

This is implemented in `app_test_setup.dart → startApp()`:
```dart
const _qaRefreshToken = String.fromEnvironment('QA_REFRESH_TOKEN');
const _qaSubdomain   = String.fromEnvironment('QA_SUBDOMAIN');
const _qaAccessToken = String.fromEnvironment('QA_ACCESS_TOKEN'); // optional
```

Only `QA_REFRESH_TOKEN` + `QA_SUBDOMAIN` are strictly required. The access token is auto-refreshed.

### iOS Keychain behavior — CRITICAL

**The iOS Keychain survives app reinstalls.** This is standard iOS security behavior — Keychain belongs to the device/Simulator, not the app data container. When `flutter test` reinstalls the app, the Keychain is NOT cleared.

**Practical implication**: If you logged in via `flutter run` and the token is still valid, tests will run authenticated on the very next `flutter test` — without any `--dart-define`. No need to re-inject tokens unless the token has expired.

**When tests run unauthenticated**: the token has expired (server returns 400 on refresh), NOT because the Keychain was cleared. Renew the token using the process below.

### Rotating refresh tokens — CRITICAL

Syncro uses **rotating refresh tokens**: each call to the OAuth refresh endpoint invalidates the old refresh token and returns a new one, which the app stores in the Keychain.

**The trap**: if `startApp()` overwrites the Keychain with the dart-define token on EVERY test run, test 1 consumes `REFRESH_TOKEN_A` → server writes `REFRESH_TOKEN_B` to Keychain → test 2 overwrites Keychain with `REFRESH_TOKEN_A` again → server returns 400 `invalid_grant` → all subsequent tests run unauthenticated.

**The fix** (in `app_test_setup.dart`): `startApp()` only seeds the Keychain when it is **empty**. If a token already exists, it is left untouched — the server-rotated token from test N propagates naturally to test N+1 via the Keychain.

### Android — token rotation trap, and a corrected claim about data wipes

**Correction (2026-08-25)**: this doc previously stated flat-out that `flutter test`
"wipes [Android app data] on every reinstall." That is **not universally true** —
confirmed on a physical Android 15 device that a session created via `flutter run`
survived a subsequent `flutter test` reinstall (same APK signature, same version) and
was auto-detected by `get_qa_token.dart` within ~10 seconds. Whether data survives may
depend on APK signature/versionCode matching between the two installs, or on
device/OS-version specifics — don't treat either behavior as an absolute rule. Verify
per-device with the bootstrap sequence below rather than assuming.

What *is* still true: Android has **no OS-level Keychain-equivalent** that survives
reliably the way iOS's does, so there's no "seed only if empty" escape hatch to lean on
by default. A single static cached refresh token reused across N sequential
`flutter test` invocations (e.g. via `run_tests.sh`) can still be rejected on test 2
onward if test 1 already rotated it server-side (`invalid_grant`).

**The correct pattern** (see `run_tests.sh`): after each test file finishes, pull the
freshly-rotated token off the device before launching the next one:
```bash
adb -s "$DEVICE" exec-out "run-as $ANDROID_PACKAGE cat /data/data/$ANDROID_PACKAGE/files/patrol_token.txt"
```
Feed the extracted token into the next `--dart-define` instead of reusing the
original cached pair. Running tests one-off with a stale cached pair (e.g. copy-pasted
from a previous session's `.qa_tokens`) reproduces the same trap. The `QA_ACCESS_TOKEN`
that `run_tests.sh` describes as a "non-expiring API key" is **not guaranteed to still
be valid** — it can go stale/revoked between sessions. Don't trust that comment blindly;
verify (see silent-failure note below) before assuming a cached token still works.

**iOS and Android tokens are NOT interchangeable.** Seeding an Android run with the
cached `QA_IOS_ACCESS_TOKEN`/`QA_IOS_REFRESH_TOKEN` fails with
`{"error":"invalid_grant", ...}` even when those iOS tokens are actively working on iOS
runs in the same session. Don't use this as a shortcut — go straight to the bootstrap
sequence below.

**Manual login does NOT work under `flutter test`** — despite what `get_qa_token.dart`'s
own docstring says. Confirmed with hard evidence on a physical device: neither a real
finger tap nor a synthetic `adb shell input tap <x> <y>` at the exact "Sign In" button
coordinates triggered the button's `onPressed` while a `flutter test` run was active.
The test log instead printed `IntegrationTestWidgetsFlutterBinding`'s built-in debug
line (`Some possible finders for the widgets at Offset(x, y): find.text('Sign In') ...`)
— it detects and reports the touch location for debugging, but does **not** dispatch it
as a real gesture into the app's gesture-recognition pipeline. No overlay, no logcat
error, no permission dialog was involved — verified via screenshot + logcat. This
matches (and the docstring should be corrected to match) the "Known Limitations" section
below: the initial login must be done via `flutter run`, never `flutter test`.

**Bootstrapping a fresh Android QA token** (first time ever, or whenever the cached one
goes 401 `invalid_token` / `"Failed to authenticate account"`):
1. `fvm flutter run --flavor qa -d <device>` — a normal debug run, **not** `flutter
   test`. Log in manually here; real taps work fine under `flutter run`.
2. Confirm authenticated by grepping the run log for `/api/v1/me` → `statusCode: 200`.
3. Stop the `flutter run` process.
4. `fvm flutter test integration_test/patrol/utils/get_qa_token.dart --flavor qa -d
   <device> --dart-define=PATROL=true` — no `QA_ACCESS_TOKEN` dart-define (this is
   "first-run" mode in the script's own logic: it waits up to 3 minutes for
   `isAuthenticated($)` to become true, without needing a fresh manual tap). If the
   session survives the reinstall (see correction above), this auto-detects the
   authenticated state within seconds and prints fresh
   `QA_ACCESS_TOKEN`/`QA_REFRESH_TOKEN`/`QA_SUBDOMAIN`.
5. Save the printed tokens into `integration_test/patrol/.qa_tokens` under
   `QA_ACCESS_TOKEN=` / `QA_ANDROID_REFRESH_TOKEN=` (the file is shared with iOS — leave
   the `QA_IOS_*` and `QA_SUBDOMAIN` lines untouched; see `run_tests.sh`'s read/write
   logic for the exact format).

**The silent failure mode — verify results, don't trust exit code alone**: when a test
hits `invalid_grant`/`invalid_token`, it lands on the login screen, `isUnauthenticated($)`
returns `true`, and if the test uses the soft early-return
(`if (isUnauthenticated($)) return;`) it reports **"All tests passed!" without ever
exercising its real assertions** — a false positive that looks identical to a genuine
pass in the summary output. Confirmed twice now:
- SE-13314's Ticket Timer suite (iOS): a run with a stale cached token showed
  `invalid_grant` in the log immediately before a "passed" test that, on inspection, had
  none of its own debug print lines (e.g. `[TIMER-TEST] nav: ...`) anywhere in the
  output — proof it never left the login screen.
- A full Android run (2026-08-25) against a since-revoked cached `QA_ACCESS_TOKEN`:
  every request in the log showed `401` / `{"error":"Failed to authenticate account"}`,
  yet the suite printed `All tests passed!`.

Always grep the log for `401`/`invalid_grant`/`invalid_token` and the test's own
nav/debug markers when a shared or cached token is used — exit code 0 or "All tests
passed!" alone is not sufficient evidence the test ran for real.

### Getting / renewing QA tokens

1. Log in to the QA app manually:
   ```bash
   cd syncro-flutter
   fvm flutter run --flavor qa -d <SIMULATOR_DEVICE_ID>
   ```
   Log in through the OAuth WebView in the Simulator.

2. Run `get_qa_token.dart` (the app will auto-authenticate from the Keychain token — no manual action needed in the Simulator):
   ```bash
   fvm flutter test \
     integration_test/patrol/utils/get_qa_token.dart \
     --flavor qa \
     -d <SIMULATOR_DEVICE_ID> \
     --reporter expanded
   ```

3. Copy the output:
   ```
   ════════════════════════════════════════════
   QA TOKENS — add these to your test command:
   ════════════════════════════════════════════
   --dart-define=QA_REFRESH_TOKEN=<value>
   --dart-define=QA_SUBDOMAIN=<value>
   ════════════════════════════════════════════
   ```

### QA environment details

| Key | Value |
|-----|-------|
| Subdomain | `ballastlanedev` |
| Base URL | `https://ballastlanedev.syncromsp.com` |
| Flavor | `qa` |

---

## Test Structure

### One test per file

Each file contains exactly one `patrolWidgetTest`. Multiple tests in one file share a widget tree and cause state contamination.

### Standard test skeleton

```dart
void main() {
  setupIntegrationTest();

  patrolWidgetTest(
    'feature — what it tests',
    config: const PatrolTesterConfig(
      settleTimeout: Duration(seconds: 60),
      visibleTimeout: Duration(seconds: 30),
    ),
    ($) async {
      await withSocketFilter(() async {
        await startApp($);

        if (isUnauthenticated($)) {
          // Unauthenticated path — assert login screen
          expect($(find.text('Sign In')).exists, isTrue);
        } else {
          // Authenticated path — test the feature
        }
      });
    },
  );
}
```

### Mandatory wrappers

| Wrapper | Purpose |
|---------|---------|
| `withSocketFilter(() async { ... })` | Suppresses Phoenix WebSocket heartbeat errors that cause spurious test failures |
| `await startApp($)` | Launches `app.main()` once (guarded), seeds QA token if provided, waits for home or login screen |

**Always wrap the body in `withSocketFilter`. Always call `startApp($)` first. Never call `app.main()` directly.**

---

## Adaptive Auth Pattern

Tests must handle both authenticated and unauthenticated states. QA tokens expire.

```dart
if (isUnauthenticated($)) {
  // Minimal assertion — verify login screen is shown correctly
  expect($(find.text('Sign In')).exists, isTrue);
} else {
  // Full test path for authenticated state
  await $(find.text('Tickets')).tap();
  // ...
}
```

For features that only make sense when authenticated, use a soft early return:
```dart
if (isUnauthenticated($)) return; // skip silently — token expired
```

---

## Navigation Patterns

### Back navigation

```dart
// Normal screens (back button arrow)
await $(find.byIcon(Icons.arrow_back_ios)).tap();

// Screens with showOnBackDialog=true (AppScaffold confirmation)
await dismissWithLeaveDialog($);
// Which does: tap Icons.close → tap 'Leave Screen' in the dialog
```

### Soft guards for optional data

Use soft guards whenever content depends on QA data that may not exist:
```dart
if (!$(find.text('Some Feature')).exists) return; // no QA data — skip
```

### Finding non-labeled buttons

```dart
// Last InkWell in AppBar (e.g. "add canned response" button)
final appBar = find.byType(AppBar);
final inkwells = find.descendant(of: appBar, matching: find.byType(InkWell));
await $(inkwells.last).tap();

// Specific icon button
await $(find.byIcon(Icons.add)).tap();

// Send button in chat (IconButton with onPressed)
await $(find.byWidgetPredicate((w) => w is IconButton && w.onPressed != null)).last.tap();

// Custom keyed button
await $(find.byKey(const Key('custom_search_delegate_trailing_button'))).tap();
```

---

## Known Flaky Tests

### `ticket_detail_bottomsheet_mutations_test.dart` — Status bottom sheet ListTile timeout

Confirmed via two consecutive isolated runs (2026-08-20, iOS Simulator, QA env) failing at the exact same point: tapping the first `ListTile` in the "Select a Status" bottom sheet times out after 30s with `WaitUntilVisibleTimeoutException` ("did not find any visible (i.e. hit-testable) widgets"). Failure occurs at the Status mutation step, well before this test's own back-navigation section — unrelated to SE-13511's search-preservation changes (confirmed: the test's earlier search→select→Ticket Detail navigation succeeds every time; only the bottom-sheet tap fails). Root cause not yet investigated (candidates: QA data drift on the seeded `[TEST] E2E ticket create full` ticket's current status, or a genuine timing/animation issue with this specific bottom sheet). Needs its own investigation — not blocking SE-13511.

## Known Limitations

### System-browser-tab OAuth login (SE-13227)

As of SE-13227, the login flow no longer uses an in-app WebView — it launches the OS's own browser chrome via `flutter_web_auth_2` (Android Custom Tabs / iOS `ASWebAuthenticationSession`). This is native, out-of-process UI that `patrol_finders` cannot see or drive at all — not even the "invisible HTML inputs inside a WebView" situation the old flow had. This means:

- **Cannot test wrong-credentials error state** via integration test
- **Cannot automate the initial login** — must be done manually via `flutter run` (or seeded via a Keychain-backed token, per `app_test_setup.dart`, which is how the rest of this suite currently avoids the login screen entirely)
- Alternative for wrong-credentials: unit test `LoginCubit` with a mocked OAuth repository (see `login_cubit_test.dart`)

Driving the system browser chrome (if ever needed) would require full `patrol` (not `patrol_finders`) with native UIAutomator/XCTest bridges — see `docs/kb-projects/syncro-flutter/technical/integrations/oauth.md` for the current flow and its known test-coverage gaps.

### FlutterMentions text input

Ticket notes use `FlutterMentions`, which wraps `TextField` in a custom widget. Use index-based finding:
```dart
// Index 0 = FlutterMentions inner TextField (note body)
await $(find.byType(TextField)).at(0).enterText('Test note content');
```

### iOS Simulator Keychain location

The debug keychain on Simulator is at:
```
~/Library/Developer/CoreSimulator/Devices/{UDID}/data/Library/Keychains/keychain-2-debug.db
```
(NOT `keychain-2.db` — that file is always empty for debug builds.)

The data is encrypted by the Keychain subsystem and cannot be read directly via SQLite. Use `get_qa_token.dart` to extract tokens while the app is running.

---

## Shared Utilities Reference (`app_test_setup.dart`)

| Function | Signature | Description |
|----------|-----------|-------------|
| `setupIntegrationTest()` | `void` | Call once at top of `main()`. Initializes the integration test binding. |
| `startApp($)` | `Future<void>` | Seeds token if dart-defines provided, calls `app.main()`, polls until home or login screen. |
| `isAuthenticated($)` | `bool` | Returns `true` if "Tickets" tab is visible (home screen). |
| `isUnauthenticated($)` | `bool` | Returns `true` if "Sign In" text is visible (login screen). |
| `withSocketFilter(body)` | `Future<void>` | Suppresses Phoenix WebSocket noise during test body. |
| `dismissWithLeaveDialog($)` | `Future<void>` | Handles `showOnBackDialog=true` screens: taps close icon → confirms "Leave Screen". |
