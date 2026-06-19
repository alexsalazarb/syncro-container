# Patrol Integration Tests

**Last Updated**: 2026-06-19  
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

## Known Limitations

### WebView-based OAuth login

The login flow opens a native WebView (`webview_flutter`). HTML inputs inside the WebView are invisible to the Flutter widget tree — `patrolWidgetTest` cannot fill email/password fields or trigger login. This means:

- **Cannot test wrong-credentials error state** via integration test
- **Cannot automate the initial login** — must be done manually via `flutter run`
- Alternative for wrong-credentials: unit test `LoginCubit` with a mocked OAuth repository

To test WebView interactions in the future, use full `patrol` (not `patrol_finders`) with native UIAutomator/XCTest bridges.

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
