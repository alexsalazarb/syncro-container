# Package Compatibility Audit — Findings

**Audit Date**: 2026-06-08
**Flutter target**: 3.38.7 / Dart 3.10.7
**Auditor**: claude-sonnet-4-6

---

## Summary

Of the 56 packages audited (44 runtime + 10 dev + 4 special/pinned), approximately 60% are already compatible or require only minor effort. The five Firebase packages and `flutter_local_notifications` represent the largest upgrade surface due to native SDK version bumps (Firebase iOS SDK 12 / Android SDK 34), but Flutter-level API changes are limited. The clearest blocker is `flutter_mentions` — a custom git fork last meaningfully updated in 2021 with a `<3.0.0` Dart upper bound that will prevent resolution under Dart 3.10. The `webview_flutter_wkwebview` pin can be safely lifted: the login/auth crash introduced in 3.18.0 was fixed in 3.18.3, latest is 3.26.0. `infinite_scroll_pagination` 5.x is a complete architectural rewrite requiring migration of every paginated screen.

---

## Package Audit Table

### Runtime Dependencies

| Package | Current | Target | Category | Breaking Changes / Notes |
|---------|---------|--------|----------|--------------------------|
| app_badge_plus | 1.1.6 | 1.x | 🟢 | No major bump; Dart 3 compatible |
| app_links | 6.3.3 | 7.1.1 | 🟡 | 7.0.0 requires Flutter 3.38.1, iOS 13 min; iOS UIScene lifecycle changes. 7.1.1 bumps to AGP 9/Kotlin Gradle DSL. Core API unchanged. |
| appcheck | 1.5.3 | — | 🟢 | No major bump; assume compatible |
| auto_size_text | 3.0.0 | — | 🟢 | No major bump; stable |
| cached_network_image | 3.4.1 | — | 🟢 | No major bump |
| collection | 1.19.0 | — | 🟢 | First-party Dart package; always compatible |
| connectivity_plus | 6.0.5 | 7.1.1 | 🟡 | 7.0.0: requires AGP ≥8.12.1, Gradle ≥8.13, Kotlin 2.2.0. Core API unchanged. Toolchain-only change. |
| crypto | 3.0.6 | — | 🟢 | First-party Dart package; compatible |
| dartz | 0.10.1 | — | 🟢 | No major bump; Dart 3 compatible |
| device_info_plus | 11.3.0 | 13.1.0 | 🟡 | v12: removes `AndroidDeviceInfo.serialNumber`, requires AGP ≥8.12.1, Kotlin 2.2.0. v13: lowers requirement to Dart 3.10/Flutter 3.38.1 — ideal target. Check codebase for `serialNumber` usage. |
| dio | 5.8.0+1 | — | 🟢 | No major bump; Dart 3 compatible |
| equatable | 2.0.7 | — | 🟢 | No major bump |
| file_picker | 8.3.5 | 11.0.2 | 🟡 | v9: web `pickFiles()` loads as blobs. v10: `allowCompression` deprecated (use `compressionQuality`). v11: no API breaks. Minimal effort for iOS/Android-only target. |
| firebase_analytics | 11.4.2 | 12.4.1 | 🟡 | 12.0.0: Firebase iOS SDK 12 + Android SDK 34; removes previously deprecated methods. Must upgrade in lock-step with firebase_core. Estimate: 2–4 hrs total for all Firebase. |
| firebase_core | 3.11.0 | 4.9.0 | 🟡 | 4.0.0: Firebase iOS SDK 12 + Android SDK 34. Min iOS 13 and minSdk 21 already set in 3.x. No Flutter API changes beyond SDK bumps. |
| firebase_crashlytics | 4.3.2 | 5.2.2 | 🟡 | 5.0.0: iOS SDK 12 + Android SDK 34. No Flutter-level API changes to `recordError`, `setUserIdentifier`, etc. Must upgrade with firebase_core. |
| firebase_messaging | 15.2.2 | 16.2.2 | 🟡 | 16.0.0: iOS SDK 12 + Android SDK 34; removes long-deprecated functions (`iOSNotificationSettings`, `requestNotificationPermissions()`, `deleteInstanceID()`). Check for any remaining old-style calls. |
| firebase_remote_config | 5.4.0 | 6.5.1 | 🟡 | 6.0.0: iOS SDK 12 + Android SDK 34. API unchanged beyond SDK bumps. Must upgrade with firebase_core. |
| flutter_bloc | 9.0.1 | — | 🟢 | Already on 9.x, Dart 3 compatible. No upgrade needed. |
| flutter_cache_manager | 3.4.1 | — | 🟢 | No major bump |
| flutter_dotenv | 5.2.1 | 6.0.1 | 🟡 | 6.0.0: `clean()` now sets `isInitialized = false` (was `true`); `testLoad()` renamed to `loadFromString()`. Low impact if `clean()` not used in production. |
| flutter_foreground_task | 8.17.0 | 9.2.2 | 🟡 | 9.0.0: min Flutter 3.22/Dart 3.4; Kotlin 1.7→1.9, Gradle 7.3→8.6. No Flutter API changes. |
| flutter_local_notifications | 18.0.1 | 21.0.0 | 🔴 | THREE major versions of breaking changes. v19: removes `uiLocalNotificationDateInterpretation` from `zonedSchedule()`. v20: ALL positional params → named params in `initialize()`, `show()`, `periodicallyShow()`, `cancel()`, `zonedSchedule()`, `startForegroundService()` and more. v21: min Flutter 3.38.1/Dart 3.10.0. High call-site impact. Estimate: 0.5–1 day. |
| flutter_mentions | git fork | — | 🔴 | **BLOCKER** — See Special Cases. |
| flutter_native_splash | 2.4.4 | — | 🟢 | No major bump; actively maintained |
| flutter_secure_storage | 9.2.4 | 10.3.1 | 🟡 | 10.x: Android default cipher changed (auto-migrated with `migrateOnAlgorithmChange: true`); `encryptedSharedPreferences` deprecated; min Android SDK 23 (was 19); iOS min 12 (was 9); Windows storage now file-based. Estimate: 2–4 hrs. |
| flutter_slidable | 4.0.3 | 4.0.3 (latest) | 🟢 | Already on latest. Compatible with Flutter 3.22+. |
| flutter_svg | 2.0.17 | — | 🟢 | No major bump |
| flutter_timezone | 4.1.0 | 5.1.0 | 🟡 | 5.0.0: return type changed from `String` to `TimezoneInfo`. All call sites: `await FlutterTimezone.getLocalTimezone()` now returns `TimezoneInfo` — use `.name` to get the string. |
| get_it | 8.0.3 | 9.2.1 | 🟡 | 9.0.0: `strictDisposalOrder` parameter removed from `reset()`, `resetScope()`, `popScope()`, `popScopesTill()`, `dropScope()`. LIFO now always enforced. Remove any uses of that parameter. Low impact. |
| go_router | 14.8.0 | 17.2.3 | 🟡 | v15: URLs now case-sensitive by default (add `caseSensitive: false` to `GoRouter()` to preserve old behavior). v16: `GoRouteData` methods (`.go()`, `.push()`, `.location`) defined on class itself, requires `go_router_builder ≥ 3.0.0` if type-safe routing is used; URL case fix for branch routes. v17: `ShellRoute` observer notification changed, new `notifyRootObserver` param. No deep routing architecture changes. Estimate: 4–8 hrs. |
| grouped_list | 6.0.0 | — | 🟢 | No major bump |
| hive | 2.2.3 | — | 🟡 | Published 3 years ago. SDK constraint `>=2.12.0 <3.0.0` but pub re-interprets this as `<4.0.0` for pub.dev packages with null-safe lower bounds. Should resolve under Dart 3.10 but **must be verified** with `flutter pub get`. `hive 4.0.0-dev` is a breaking rewrite in progress. |
| hive_flutter | 1.1.0 | — | 🟡 | Same concern as `hive`. Needs `flutter pub get` verification. |
| http | 1.3.0 | — | 🟢 | First-party Dart package; compatible |
| image_picker | 1.1.2 | — | 🟢 | No major bump; actively maintained |
| infinite_scroll_pagination | 4.1.0 | 5.1.1 | 🔴 | 5.0.0 is a **complete architectural rewrite**. Removed: `appendPage()`, `appendLastPage()`, `retryLastFailedRequest()`, `addPageRequestListener()`, `firstPageKey`, `itemList`/`error`/`nextPageKey` getters/setters. Now uses `PagingState.copyWith()`. `PagedLayoutBuilder` no longer accepts `pagingController` — takes `PagingState` + `fetchNextPage`. New `PagingListener` widget required. Every paginated screen must be rewritten. Estimate: 0.5–1 day. |
| intl | 0.19.0 | 0.20.2 | 🟢 | No breaking API changes 0.19→0.20. CLDR updated to v46. Dependency cleanup. Safe to upgrade. |
| local_auth | any | 3.0.1 | 🟡 | 3.0.0: `stickyAuth` → `persistAcrossBackgrounding`; `useErrorDialogs` removed; throws `LocalAuthException` (not `PlatformException`); min Android API 24, iOS 13. Must pin constraint and update call sites. Estimate: 1–2 hrs. |
| mobile_scanner | 7.1.4 | 7.2.0 (latest) | 🟢 | Already on 7.x (Dart 3 compatible). Patch upgrade to 7.2.0, no breaking changes. |
| modal_bottom_sheet | 3.0.0 | 3.0.0 (latest) | 🟡 | Last published 2 years ago (targets Flutter 3.19). No update for Flutter 3.38. **Needs manual `flutter pub get` + runtime test.** Community reports suggest likely OK, but unverified. |
| oauth_webauth | 5.1.0 | 5.2.0 | 🟢 | 5.2.0: no breaking changes. Dart 3+ compatible since 4.0.1+2. |
| overlay_support | 2.1.0 | 2.1.0 (latest) | 🟡 | Last published 3 years ago. No Flutter 3.38 verification. Likely resolves via pub reinterpretation, but **needs smoke test**. |
| package_info_plus | 8.2.1 | 9.0.1 | 🟡 | 9.0.0: AGP ≥8.12.1, Gradle ≥8.13, Kotlin 2.2.0. No API changes. Toolchain-only. |
| path_provider | 2.1.5 | — | 🟢 | First-party Flutter package; compatible |
| pendo_sdk | 3.7.1 | 3.13.2 | 🟡 | No documented breaking changes 3.7→3.13. Feature additions only (Session Replay GA in 3.9.0). Latest published 6 days from audit date. SDK constraint for Dart 3.10 needs verification. |
| permission_handler | 11.3.1 | 12.0.2 | 🟡 | 12.0.0: only change is Android compileSdk 35 and `permission_handler_android` 13.0.0. No API changes. Toolchain-only. |
| phoenix_socket | 0.7.6 | 0.8.0 | 🟢 | 0.8.0 adds binary message support; no breaking changes. Dart 3 compatible. |
| photo_view | 0.15.0 | — | 🟢 | No major bump; maintained |
| restart_app | 1.3.2 | — | 🟢 | No major bump |
| scribble | 0.10.0+1 | 0.10.0+1 (latest) | 🔴 | Last published **2 years ago** (whynotmake.it). SDK constraint includes `<3.0.0`. Unlike pub.dev packages, resolution behavior under Dart 3.10 is uncertain. Package uses `value_notifier_tools` internals. No maintenance activity. If resolution fails or `flutter analyze` shows errors, a replacement for the drawing/signature feature must be found. **Needs hands-on `flutter pub get` + `flutter analyze` test.** |
| shared_preferences | 2.5.2 | — | 🟢 | First-party Flutter package; compatible |
| shimmer | 3.0.0 | — | 🟢 | No major bump |
| timeago | 3.7.0 | — | 🟢 | No major bump |
| timezone | 0.9.4 | 0.11.0 | 🟡 | 0.11.0: `Location.offset` changed from `int` to `Duration`. Update uses to `.offset.inSeconds` or `.inHours`. Low impact unless offset is accessed directly. |
| url_launcher | 6.3.1 | — | 🟢 | First-party Flutter package; compatible |
| visibility_detector | 0.4.0+2 | — | 🟢 | No major bump |
| webview_flutter | 4.10.0 | — | 🟢 | No major bump |
| webview_flutter_android | 4.10.0 | — | 🟢 | No major bump |
| webview_flutter_wkwebview | ^3.16.0 (pinned) | 3.26.0 | 🟡 | See Special Cases. Pin can be safely lifted. |

### Dev Dependencies

| Package | Current | Target | Category | Notes |
|---------|---------|--------|----------|-------|
| bloc_test | 10.0.0 | — | 🟢 | Dart 3 compatible |
| build_runner | 2.4.13 | — | 🟢 | `build_resolvers` and `build_runner_core` functionality consolidated into `build_runner`. No action needed. |
| firebase_core_platform_interface | 5.4.0 | — | 🟢 | Follows firebase_core versioning |
| firebase_crashlytics_platform_interface | 3.8.2 | — | 🟢 | Same |
| flutter_launcher_icons | 0.14.3 | — | 🟢 | No major bump |
| flutter_lints | 5.0.0 | — | 🟢 | Compatible |
| hive_generator | 2.0.1 | — | 🟡 | Same concern as `hive` — old, likely resolves but needs verification |
| mocktail | 1.0.4 | — | 🟢 | Dart 3 compatible |
| mockito | 5.4.4 | — | 🟢 | Dart 3 compatible |
| test | 1.25.7 | — | 🟢 | First-party Dart package; compatible |

---

## Special Cases

### flutter_mentions (Custom Fork) — BLOCKER

**Repository**: `github.com/alexsalazarb/flutter_mentions`, branch `syncro_update`

**Last meaningful commit**: May 2021 (null-safety migration, version 2.0.1). One minor commit in January 2025 (`Sort key list to prioritize mentions`).

**SDK constraint in fork's pubspec.yaml**: `sdk: '>=2.12.0 <3.0.0'`

This is the critical issue. The `<3.0.0` upper bound means `flutter pub get` under Dart 3.10 will fail to resolve this git-sourced dependency. Unlike pub.dev packages where pub applies a compatibility re-interpretation hack, git dependencies are resolved with their literal constraints.

The original `flutter_mentions` on pub.dev (v2.0.1, by fayeed) has the same constraint and is also unmaintained.

**Options (in order of effort)**:

1. **Update the fork** *(recommended)*: Update `pubspec.yaml` SDK constraint to `>=3.0.0 <4.0.0`, update `flutter_portal: ^0.4.0` to its current version, run `dart fix --apply`, verify compilation. The core Dart code (annotation editing, mention detection) uses standard Flutter APIs with no Dart 3-incompatible constructs. Estimated effort: **2–4 hours**.

2. **Vendor the code**: Copy the fork's source files into `packages/flutter_mentions/` in the app repo, remove the git dependency, pin as a local path dep. Same code changes as option 1 but no external git dependency. Estimated effort: **2–4 hours**.

3. **Migrate to a maintained alternative**: `mention_tag_text_field` or `flutter_quill` with mention support. Requires more extensive UI migration. Estimated effort: **1–2 days**.

**Recommendation**: Option 1. The fork has custom logic (`syncro_update` branch) that would be lost in option 3.

---

### webview_flutter_wkwebview (Pinned to ^3.16.0)

**Background**: The pubspec comment `# 3.18.1 make login fails` refers to a crash introduced in 3.18.0 where authentication/OAuth challenges crashed iOS WebView with `Could not cast value of type 'NSNull' to 'AuthenticationChallengeResponse'` (GitHub issue #162437).

**Fix status**: **Version 3.18.3** explicitly fixed "a crash where the native `AuthenticationChallengeResponse` could not be found for auth requests."

**Current latest**: 3.26.0 (published 4 days before audit date).

**Recommendation**: Change the pin from `^3.16.0` to `^3.18.3` (conservative) or `^3.26.0` (recommended). Before release, manually test the OAuth/login flow on a physical iOS device to confirm.

---

### local_auth (any constraint)

**Resolved version**: With `any`, pub resolves to **3.0.1** (latest).

**Breaking changes in 3.0.0**:
- `authenticate()` param `stickyAuth` → `persistAcrossBackgrounding`; `useErrorDialogs` removed
- Throws `LocalAuthException` with typed `LocalAuthExceptionCode` instead of `PlatformException`
- Min Android API 24 (was 16), iOS 13

**Recommendation**: Pin to `^3.0.1`. Update `authenticate()` call sites and exception catch blocks. Estimated effort: 1–2 hours.

---

## Transitive Discontinued Packages

| Package | Status | Action |
|---------|--------|--------|
| `js` (discontinued Feb 2025) | Transitive dep of older packages; replaced by `dart:js_interop` / `package:web`. Current versions of all direct deps in this pubspec have already migrated. | None at app level — resolved by upgrading direct deps. |
| `build_resolvers` (discontinued) | Functionality merged into `build_runner` itself in 2024. | None — `build_runner ^2.4.13` already includes it. |
| `build_runner_core` (discontinued) | Same as above — merged into `build_runner`. | None. |

---

## Upgrade Effort Estimate

| Category | Count | Est. Effort | Key packages |
|----------|-------|-------------|--------------|
| 🟢 Green | 29 | 0 days | All first-party packages, `flutter_bloc`, `dio`, `intl`, `mobile_scanner`, `oauth_webauth`, `phoenix_socket`, and ~20 others |
| 🟡 Yellow | 24 | 3–5 days | All 5 Firebase packages (0.5–1 day together), `go_router` (4–8 hrs), `flutter_local_notifications`*, `infinite_scroll_pagination`*, `local_auth`, `flutter_timezone`, `flutter_secure_storage`, `get_it`, toolchain-only bumps (AGP/Gradle packages) |
| 🔴 Red | 3 | 2–4 days | `flutter_mentions` (1 day, blocker must resolve first), `scribble` (unknown, needs triage) |
| **Total** | **56** | **5–9 days** | |

*`flutter_local_notifications` and `infinite_scroll_pagination` are categorized Yellow by some measures (solvable) but are the heaviest Yellow items.

**Firebase batch note**: All 5 Firebase packages must be upgraded in a single batch (they share the same native SDK bump). They should be treated as 1 unit of ~0.5–1 day of work including testing.

---

## Blockers (Kill Criteria)

### Must resolve before upgrade can proceed

1. **`flutter_mentions` fork** — `<3.0.0` SDK constraint will fail `flutter pub get` under Dart 3.10 as a git dependency. Fix the fork first; everything else is unblocked after this.

### Significant effort but solvable

2. **`flutter_local_notifications` 18→21** — Every notification call site requires positional→named param conversion across 3 major versions. Estimate: 0.5–1 day.

3. **`infinite_scroll_pagination` 4→5** — Every paginated screen requires architectural migration. `PagingController` API is fundamentally different. Estimate: 0.5–1 day.

4. **`scribble` 0.10.0+1** — Unmaintained 2 years; `<3.0.0` bound. Must test with `flutter pub get` + `flutter analyze`. If it fails, a replacement for the drawing/signature feature must be sourced before upgrade.

### Needs hands-on verification

5. **`hive` + `hive_flutter`** — Old packages; should resolve via pub reinterpretation but must run `flutter pub get` to confirm.
6. **`modal_bottom_sheet`** — 2 years without update, targets Flutter 3.19. May have internal Flutter API compatibility issues.
7. **`overlay_support`** — 3 years without update. Same concern.

---

## Sources

- [flutter_mentions fork pubspec.yaml (syncro_update)](https://api.github.com/repos/alexsalazarb/flutter_mentions/contents/pubspec.yaml?ref=syncro_update): confirmed `<3.0.0` SDK constraint
- [flutter_mentions commit log (syncro_update)](https://api.github.com/repos/alexsalazarb/flutter_mentions/commits?sha=syncro_update): last meaningful commit May 2021
- [flutter/flutter issue #162437: webview crash on iOS after 3.18.0](https://github.com/flutter/flutter/issues/162437): confirms login crash and fix
- [webview_flutter_wkwebview changelog](https://pub.dev/packages/webview_flutter_wkwebview/changelog): 3.18.3 fixes auth crash; latest 3.26.0
- [firebase_core changelog](https://pub.dev/packages/firebase_core/changelog): 4.0.0 Firebase iOS SDK 12 + Android SDK 34
- [firebase_messaging changelog](https://pub.dev/packages/firebase_messaging/changelog): 16.0.0 SDK bumps + deprecated removal
- [firebase_crashlytics changelog](https://pub.dev/packages/firebase_crashlytics/changelog): 5.0.0 SDK bumps
- [firebase_analytics changelog](https://pub.dev/packages/firebase_analytics/changelog): 12.0.0 SDK bumps + deprecated removal
- [firebase_remote_config changelog](https://pub.dev/packages/firebase_remote_config/changelog): 6.0.0 SDK bumps
- [FlutterFire migration guide](https://firebase.flutter.dev/docs/migration/)
- [go_router changelog](https://pub.dev/packages/go_router/changelog): v15 case-sensitive URLs; v16 GoRouteData; v17 ShellRoute observers
- [flutter_local_notifications changelog](https://pub.dev/packages/flutter_local_notifications/changelog): v19–v21 breaking changes
- [infinite_scroll_pagination changelog](https://pub.dev/packages/infinite_scroll_pagination/changelog): v5 architectural rewrite
- [local_auth changelog](https://pub.dev/packages/local_auth/changelog): v3 exception type + API changes
- [get_it changelog](https://pub.dev/packages/get_it/changelog): v9 LIFO + strictDisposalOrder removed
- [flutter_timezone changelog](https://pub.dev/packages/flutter_timezone/changelog): v5 returns TimezoneInfo not String
- [timezone changelog](https://pub.dev/packages/timezone/changelog): 0.11.0 Location.offset is Duration
- [flutter_secure_storage changelog](https://pub.dev/packages/flutter_secure_storage/changelog): 10.x cipher changes
- [device_info_plus changelog](https://pub.dev/packages/device_info_plus/changelog): v12 removes serialNumber; v13 targets Dart 3.10
- [connectivity_plus changelog](https://pub.dev/packages/connectivity_plus/changelog): v7 toolchain update only
- [permission_handler changelog](https://pub.dev/packages/permission_handler/changelog): v12 compileSdk 35 only
- [app_links changelog](https://pub.dev/packages/app_links/changelog): v7 Flutter 3.38.1 + iOS UIScene
- [flutter_dotenv changelog](https://pub.dev/packages/flutter_dotenv/changelog): v6 clean() resets isInitialized
- [file_picker changelog](https://pub.dev/packages/file_picker/changelog): v9–v11
- [package_info_plus changelog](https://pub.dev/packages/package_info_plus/changelog): v9 toolchain only
- [flutter_foreground_task changelog](https://pub.dev/packages/flutter_foreground_task/changelog): v9 SDK bump
- [pendo_sdk changelog](https://pub.dev/packages/pendo_sdk/changelog): 3.7→3.13 no breaking changes
- [build_runner — discontinued sub-packages merged in](https://pub.dev/packages/build_runner)
- [js discontinued — dart-lang/build issue #3861](https://github.com/dart-lang/build/issues/3861)
- [Flutter 3.38 release notes](https://docs.flutter.dev/release/release-notes/release-notes-3.38.0)
