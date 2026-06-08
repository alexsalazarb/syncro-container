# Upgrade Viability Report — Flutter 3.32.4 → 3.38.7

**Report Date**: 2026-06-08
**JIRA**: SE-12530
**Author**: Claude Sonnet 4.6 (flutter-upgrade-viability assessment)

---

## Executive Summary

**CONDITIONAL GO** — The upgrade is viable and recommended, but must not start until `flutter_mentions` (the internal git fork) has its SDK constraint updated from `<3.0.0` to `<4.0.0`. With that single blocker resolved (2–4h), the path is clear: 3 red packages, 24 yellow packages, and one manual iOS migration account for an estimated **8–14 days** of total upgrade work. No kill criteria were triggered: the `flutter_mentions` fork is fixable without maintainer involvement, no critical package requires more than 2 weeks of work, and iOS/Android min targets remain satisfied.

---

## Upgrade Scope

- **Flutter SDK**: 3.32.4 → 3.38.7 (Dart 3.8.1 → 3.10.7), crossing Flutter 3.35 and 3.38 stable releases
- **Packages audited**: 56 (29 🟢 green, 24 🟡 yellow, 3 🔴 red)
- **SDK breaking changes requiring code work**: 6 (UISceneDelegate migration, SnackBar persistence change, Android predictive-back, deprecation sweep, SystemChrome orientation on API 36, Android abiFilters)
- **Blockers**: 1 — `flutter_mentions` git fork has a literal `<3.0.0` SDK constraint that will fail `flutter pub get` under Dart 3.10

---

## Blockers

### BLOCKER — flutter_mentions git fork: SDK constraint `<3.0.0`

Unlike pub.dev packages (which use reinterpretation), git dependencies use their literal pubspec.yaml constraint. The current fork at `github.com/alexsalazarb/flutter_mentions` (ref: `syncro_update`) declares `sdk: '>=2.12.0 <3.0.0'`, which is incompatible with Dart 3.10.7.

**Resolution (must be done first, before any other upgrade step):**

1. Clone / checkout the fork locally
2. Update `pubspec.yaml`: `sdk: '>=3.0.0 <4.0.0'`
3. Update `flutter_portal` dependency to a Dart 3-compatible version
4. Run `dart fix --apply` (Dart source itself is Dart-3 compatible — no logic changes expected)
5. Commit and push to the fork's `syncro_update` branch
6. Verify: `flutter pub get` resolves cleanly in the main app

**Effort**: 2–4 hours. This unblocks all subsequent steps.

---

## Effort Estimate

| Work item | Effort |
|-----------|--------|
| **BLOCKER** flutter_mentions fork: update SDK constraint + dart fix | 2–4 h |
| UISceneDelegate iOS migration (AppDelegate + SceneDelegate + Info.plist) | 2–4 h |
| SnackBar auto-dismiss sweep (`persist: false` where needed) | 1–2 h |
| Deprecation sweep (AppBarTheme, DropdownButtonFormField, Switch, showCupertinoSheet) | 1–2 h |
| Android abiFilters conflict resolution | 0.5–1 h |
| SystemChrome orientation investigation (Android API 36) | 1–2 h |
| flutter_local_notifications 18 → 21 (positional → named params, v20 API, v21 min SDK) | 4–8 h |
| infinite_scroll_pagination 4 → 5 (full API rewrite across all paginated screens) | 4–8 h |
| firebase_* lock-step upgrade (5 packages, iOS SDK 12 + Android SDK 34) | 4–8 h |
| go_router 14 → 17 (case-sensitivity fix, GoRouteData, ShellRoute observer) | 4–8 h |
| local_auth upgrade (stickyAuth rename, exception type change, min API 24) | 1–2 h |
| get_it 8 → 9 (strictDisposalOrder param removal) | 1–2 h |
| flutter_secure_storage 9 → 10 (cipher migration, migrateOnAlgorithmChange) | 2–4 h |
| flutter_timezone 4 → 5 (String → TimezoneInfo.name) | < 1 h |
| Remaining yellow packages (verify + minor adjustments) | 4–8 h |
| Hands-on verification packages (hive, scribble, modal_bottom_sheet, overlay_support) | 2–4 h |
| webview_flutter_wkwebview pin lift + physical device test | 1–2 h |
| QA regression pass (full app smoke test on iOS + Android) | 1–2 days |
| **Sub-total (raw)** | **~6.5–11 days** |
| **Testing / buffer (+20%)** | **~1.5–2.5 days** |
| **Total (with buffer)** | **8–14 days** |

---

## Risk Register

| Risk | Severity | Mitigation |
|------|----------|------------|
| UISceneDelegate migration breaks Pendo tracking or screen-blur lifecycle hooks | HIGH | Implement SceneDelegate step-by-step; run Pendo integration tests before and after; verify applicationWillResignActive equivalent in SceneDelegate |
| infinite_scroll_pagination v5 rewrite misses edge cases (empty states, error states, first page retries) | HIGH | Audit all paginated screens before starting; write widget tests for each state before migrating |
| flutter_secure_storage cipher migration causes data loss for existing users | HIGH | Set `migrateOnAlgorithmChange: true` before releasing; test migration on a device with existing stored credentials |
| flutter_mentions fork `dart fix --apply` introduces regressions in mention parsing | MEDIUM | Run the app's mention-related tests after fix; manual test in the mentions input screen |
| firebase_* upgrade breaks push notifications or Crashlytics on iOS | MEDIUM | Upgrade all firebase_* packages in a single atomic commit; test on physical device immediately after |
| scribble 0.10.0+1 is unmaintained (2 years, `<3.0.0` bound) — may require replacement | MEDIUM | Evaluate replacement (e.g. perfect_freehand) before starting upgrade; if no drop-in replacement, scope a migration of the drawing feature |
| go_router v15 case-sensitivity change silently breaks deep links | MEDIUM | Add `caseSensitive: false` to all GoRoute definitions as the first step of go_router upgrade; validate all deep links via integration test |
| modal_bottom_sheet unmaintained (2 years, targets Flutter 3.19) — may fail to resolve | MEDIUM | Run `flutter pub get` immediately after SDK bump to confirm resolution; have wolt_modal_sheet as a fallback replacement |
| SystemChrome.setPreferredOrientations silently failing on Android API 36 | MEDIUM | Audit all orientation lock usages; test on emulator with API 36 target |
| Android predictive-back transitions cause visual regressions on custom routes | LOW | QA sweep on Android 14+ device; add `predictiveBackPageTransitionBuilder: null` as escape hatch if needed |

---

## Packages Needing Hands-On Verification

These packages have constraints that may resolve without code changes under `flutter pub get`, but require confirmation before assuming they are green:

| Package | Concern | Verification step |
|---------|---------|-------------------|
| `hive` + `hive_flutter` | Old constraints; likely resolves via pub reinterpretation but unverified | Run `flutter pub get`; check `flutter analyze` output |
| `scribble 0.10.0+1` | Unmaintained 2 years; `<3.0.0` bound; may fail to resolve | Run `flutter pub get`; if fails, evaluate replacement |
| `modal_bottom_sheet` | 2 years without update; targets Flutter 3.19 | Run `flutter pub get`; check compatibility shim |
| `overlay_support` | 3 years without update | Run `flutter pub get`; check `flutter analyze` output |
| `webview_flutter_wkwebview 3.16.0` | Pinned due to known auth crash; fix landed in 3.18.3 | Lift pin, test auth flows on physical iOS device |

---

## Recommended Execution Order (if GO)

### Phase 0 — Unblock (before touching pubspec.yaml in the main app)

1. **Fix flutter_mentions fork**: update SDK constraint to `>=3.0.0 <4.0.0`, update flutter_portal dep, run `dart fix --apply`, push to `syncro_update` branch.
2. Confirm: `flutter pub get` resolves cleanly with the new fork ref.

### Phase 1 — SDK version bump

3. Update `.fvmrc`: `"flutter": "3.38.7"`, run `fvm use 3.38.7`.
4. Run `flutter pub get` — expect it to succeed now that the blocker is resolved. Note any packages that fail to resolve (hands-on verification list).
5. Run `flutter analyze` — record all warnings and errors as the baseline for the deprecation sweep.

### Phase 2 — Red packages (must compile before any feature work)

6. **flutter_local_notifications 18 → 21**: update pubspec constraint, migrate all call sites to named params (v20 API), handle uiLocalNotificationDateInterpretation removal.
7. **infinite_scroll_pagination 4 → 5**: audit all paginated screens, rewrite each to use `PagingState.copyWith()` + `PagingListener`.
8. Run `flutter analyze` + `flutter test` after each red package to isolate regressions.

### Phase 3 — iOS native migration

9. **UISceneDelegate migration**: create `SceneDelegate.swift`, update `AppDelegate.swift` to use `FlutterImplicitEngineDelegate`, preserve Pendo setup and screen-blur hooks in SceneDelegate lifecycle methods, update `Info.plist` with `UIApplicationSceneManifest`.
10. Run on physical iOS device; verify Pendo tracking, URL scheme handling, and screen-blur behavior.

### Phase 4 — Yellow packages (in dependency order)

11. **firebase_* lock-step** (firebase_core → firebase_auth → firebase_messaging → firestore → crashlytics): upgrade all five in one PR; test on physical device.
12. **go_router 14 → 17**: add `caseSensitive: false` to all routes first, then handle GoRouteData and ShellRoute observer changes; validate all deep links.
13. **flutter_secure_storage 9 → 10**: add `migrateOnAlgorithmChange: true`, update to new cipher, test on device with existing stored data.
14. **local_auth**: rename `stickyAuth` → `persistAcrossBackgrounding`, update exception handling from `PlatformException` to `LocalAuthException`.
15. **flutter_timezone 4 → 5**: update all call sites from `String` return to `.name` accessor.
16. **get_it 8 → 9**: remove `strictDisposalOrder` param from `reset()`/scope calls.
17. Remaining yellow packages + hands-on verification packages (hive, scribble, modal_bottom_sheet, overlay_support, webview_flutter_wkwebview).

### Phase 5 — SDK deprecation sweep

18. Run `flutter analyze`, fix all deprecation warnings: `AppBarTheme.color → backgroundColor`, `DropdownButtonFormField.value → initialValue`, `Switch.activeColor → activeThumbColor`, `showCupertinoSheet pageBuilder → builder`.
19. **SnackBar sweep**: find all `SnackBar(action:)` usages, add `duration` or `persist: false` where auto-dismiss is expected.
20. **Android abiFilters**: review `build.gradle` for manual `abiFilters`; remove or align with Flutter's new defaults.
21. **SystemChrome orientation**: audit all `setPreferredOrientations` calls; test on Android API 36 emulator.

### Phase 6 — QA and release readiness

22. Full regression pass on iOS (physical device) and Android (physical device, API 33+ and API 36).
23. Verify Android predictive-back transitions on Android 14+ device.
24. Verify Pendo analytics session continuity across the UISceneDelegate migration.
25. Run complete test suite (`fvm flutter test`); fix any failures.
26. Submit to internal QA track; tag as upgrade verification build.

---

## Recommendation

**CONDITIONAL GO.** The upgrade from Flutter 3.32.4 to 3.38.7 is recommended. The estimated investment of **8–14 days** is proportionate to the value delivered: Dart 3.10.7 language features, Flutter 3.38 performance improvements, and mandatory compliance with the upcoming iOS 26 SDK requirement (UISceneDelegate migration, which becomes mandatory in Flutter 3.41 regardless of when this upgrade happens).

**Conditions that must be met before starting:**

1. The `flutter_mentions` fork blocker must be resolved (Phase 0) — this is the only item that will cause `flutter pub get` to fail immediately and blocks all other work.
2. `scribble` must be confirmed resolvable or a replacement scoped before committing to the timeline — if it requires a feature migration, add 1–3 days to the estimate.
3. The upgrade should be executed on a dedicated feature branch with incremental PRs per phase to minimize review surface and isolate regressions.

The upgrade is **not** a high-risk endeavor — none of the breaking changes are architecturally complex, the firebase and go_router migrations are well-documented, and the codebase already meets the new iOS/Android minimum targets. The highest-risk item (UISceneDelegate) is a native iOS migration with clear documentation and a known scope.

Do not proceed with the upgrade in parallel with other feature work on the same files. Dedicate a focused sprint.
