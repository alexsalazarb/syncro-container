# Baseline Analysis — Flutter 3.44.1 / Dart 3.12.1

**Date**: 2026-06-08
**From**: Flutter 3.32.4 / Dart 3.8.1
**To**: Flutter 3.44.1 / Dart 3.12.1
**Command**: `fvm flutter analyze` (exit 1)

---

## Versions Confirmed

| Tool | Version |
|------|---------|
| Flutter | 3.44.1 (channel stable) |
| Dart | 3.12.1 |
| DevTools | 2.57.0 |
| fvm | 3.44.1 pinned in .fvmrc |

---

## pub get Resolution Notes

Three conflicts resolved before baseline:

1. **`win32` conflict** (`file_picker 11.x` needs `^5.9`, `device_info_plus 13.x` needs `^6.0.1`)
   → Added `dependency_overrides: win32: ^6.0.1` — safe, win32 is Windows-desktop-only dep
   
2. **`firebase_core_platform_interface ^5.4.0`** too old for `firebase_remote_config ^6.5.1`
   → Bumped to `^7.0.1` (matched new Firebase major)
   
3. **`hive_generator ^2.0.1`** requires `analyzer <7.0.0`, conflicts with `test ^1.25.7` (needs `analyzer >=8.0.0`)
   → Removed `hive_generator` from dev_dependencies. Generated `.g.dart` files are committed; regen is not needed for compilation.

---

## Analyze Output Summary

Total issues: **403** (of which ~320 are noise in `build/SourcePackages/` from Firebase SPM examples)

**Issues in our codebase (`lib/` + `test/`)**: ~80 errors + ~20 infos/warnings

---

## Errors by Task

### task-02 — flutter_local_notifications (4 errors)
**File**: `lib/core/services/push_notification/notifications_manager.dart`
- Line 147: `settings` named param required (was positional)
- Line 148: Too many positional args: 0 expected
- Line 237: `id` named param required (was positional)  
- Line 238: Too many positional args: 0 expected, 4 found

### task-03 — infinite_scroll_pagination (40+ errors across 5 features)
**Files**: alerts_cubit.dart, alerts_view.dart, assets_cubit.dart, assets_view.dart, appointments_list_cubit.dart, ticket_cubit.dart, ticket_view.dart, appointments_paginated_page.dart, ticket_filter_bar.dart

Repeated patterns across all files:
- `firstPageKey` param removed from `PagingController` constructor
- `fetchPage` / `getNextPageKey` required params not provided
- `addPageRequestListener` method removed
- `appendPage` / `appendLastPage` methods removed
- `itemList` getter removed
- `.error` setter removed (now read-only)
- `retryLastFailedRequest` getter removed
- `pagingController:` param removed from `PagedListView` (replaced by `state:` + `fetchNextPage:`)

### task-05 — Firebase (test mocks, 8 errors)
**File**: `test/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer_test.dart`
- `PigeonInitializeResponse` — internal Firebase type renamed/removed
- `PigeonFirebaseOptions` — internal Firebase type renamed/removed
- `TestFirebaseCoreHostApi.setup` — method removed

→ Mocks need regeneration with `build_runner` after Firebase upgrade.

### task-07 — Yellow packages service layer (14 errors + 6 mock errors)

**flutter_timezone** (3 errors in `appointments_paginated_page.dart`):
- `TimezoneInfo` can't be assigned to `String?` (return type changed)
- `unrelated_type_equality_checks` warning on `TimezoneInfo` vs `String?`

**local_auth** (4 errors in `application_lock_cubit.dart` + `unlock_app_cubit.dart`):
- `options` param undefined — `AuthenticationOptions` class removed/renamed
- `AuthenticationOptions` isn't a class

**local_auth mock** (1 error in `test/features/general/repositories_impl_test.mocks.dart`):
- `MockLocalAuthentication.authenticate` signature mismatch (old `options:` param)

**flutter_secure_storage mock** (6 errors in `test/core/services/storage_manager_test.mocks.dart`):
- `IOSOptions` → `AppleOptions` in write/read/containsKey/delete/readAll/deleteAll
- `String?` key → `String` key (non-nullable now)
→ Mocks need regeneration with `build_runner`.

**file_picker** (1 error in `lib/features/ticket/ticket_attachment/presentation/ticket_attachment_view.dart`):
- `FilePicker.platform` getter undefined — removed in file_picker 11.x
→ **NEWLY DISCOVERED** — not in original task-07 scope, needs investigation

**flutter_secure_storage** (1 info in `lib/core/services/storage.dart`):
- `encryptedSharedPreferences` deprecated — will be removed in v11

### task-08 — SDK Deprecation Sweep (6 errors + infos)

**`lib/core/global_widgets/custom_datetime_picker/time_picker.dart`** (3 errors):
- `InputDecorationTheme` → `InputDecorationThemeData` (Flutter SDK API rename)
- `_TimePickerDefaults.inputDecorationTheme` invalid override
- `announce` deprecated → `sendAnnouncement`
→ **NEWLY DISCOVERED** — custom datetime picker copy has Flutter internal API usage

**`lib/core/utils/date_picker.dart`** (4 infos):
- `announce` deprecated → `sendAnnouncement` (×4)

**`lib/features/settings/appearance/presentation/appearance_page.dart`** (2 infos):
- `Radio.groupValue` deprecated → `RadioGroup`
- `Radio.onChanged` deprecated → `RadioGroup`

**`lib/core/usecases/usecase.dart`** (6 infos):
- Type parameter named `Type` clashes with visible type name

---

## Build/ Noise (ignore — not our code)

- `build/ios/SourcePackages/firebase_analytics-12.4.2/example/` — missing `in_app_purchase` dep in Firebase example
- `build/macos/SourcePackages/firebase_*/` — same Firebase example errors
- `build/*/SourcePackages/firebase_remote_config*/` — missing example home_page.dart

→ These are Firebase SPM example apps bundled by iOS/macOS builds. Not our code. Can be excluded from analysis by adding `build/` to `analysis_options.yaml` excludes.

---

## Swift Package Manager Warning (forward concern)

7 plugins don't support SPM for iOS yet:
`restart_app`, `pendo_sdk`, `flutter_inappwebview_ios`, `flutter_foreground_task`, `sqflite`, `appcheck`, `app_badge_plus`

Will become an error in a future Flutter version. Not blocking today.

---

## New Discoveries (not in original plan)

1. **`FilePicker.platform` removed** — `ticket_attachment_view.dart:160`. Add to task-07 scope.
2. **`custom_datetime_picker/time_picker.dart`** — has Flutter internal API (`InputDecorationTheme` → `InputDecorationThemeData`). This is a copy-pasted Flutter widget that references private framework APIs. Add to task-08 scope.
3. **Radio deprecations** (`groupValue`/`onChanged`) — `appearance_page.dart`. Add to task-08.
4. **Test mocks need regeneration** — `storage_manager_test.mocks.dart` and `repositories_impl_test.mocks.dart` need `build_runner` after task-05/07.

---

## Action Map

| Error Category | Count | Task |
|---------------|-------|------|
| flutter_local_notifications positional params | 4 | task-02 |
| infinite_scroll_pagination v5 API rewrite | ~40 | task-03 |
| Firebase test mocks (Pigeon internals) | 8 | task-05 |
| flutter_timezone TimezoneInfo return type | 3 | task-07 |
| local_auth AuthenticationOptions | 4 | task-07 |
| flutter_secure_storage IOSOptions→AppleOptions mocks | 6 | task-07 |
| FilePicker.platform removed | 1 | task-07 |
| custom_datetime_picker InputDecorationThemeData | 3 | task-08 |
| date_picker announce deprecation | 4 | task-08 |
| Radio groupValue/onChanged deprecation | 2 | task-08 |
| build_runner mock regeneration | — | task-07/05 |
| build/ SourcePackages noise | ~300 | n/a (exclude) |
