# Status: iOS — guard `NotificationManager.requestPermissions` against uncaught `firebase_messaging/unknown`

**Current Status**: complete
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: `plan/fix-production-crashes-v180/phase-2/task-03-notification-permission-crash` (syncro-flutter submodule, based on `develop`)
**PR**: N/A (PR_INTEGRATION=false) — branch pushed to `bla`, PR pending manual creation (see Artifacts)

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | complete | claude | Fix + regression test implemented, verified red-before/green-after, `flutter analyze` clean, full suite run (1 unrelated pre-existing failure noted below) |

## Summary

Confirmed the task's Context was accurate: `requestPermissions()` had no try/catch around `messaging.requestPermission(...)`. Wrapped it, matching the file's existing `logger()`-on-failure pattern; returns `false` on any exception instead of propagating.

## Regression Test

Added `test/core/services/push_notification/notifications_manager_test.dart`. Mocks `FirebaseMessaging`'s method channel (`plugins.flutter.io/firebase_messaging`, method `Messaging#requestPermission`) to throw a `PlatformException(code: 'unknown', ...)`, reproducing the exact production error. Verified manually:
- **Red (before fix)**: test threw the unhandled `[firebase_messaging/unknown] Notifications are not allowed for this application.` exception — confirmed by temporarily stashing the fix and re-running.
- **Green (after fix)**: `requestPermissions()` returns `false`, no exception.
- Added a second test for the happy path (`authorizationStatus` granted → returns `true`) using the real wire-format status code (`1`, per `firebase_messaging_platform_interface`'s `convertToAuthorizationStatus` — NOT the same as `AuthorizationStatus.authorized.index`, which would silently misrepresent as `denied`).

Following this project's existing Firebase-mocking convention (`test/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer_test.dart`'s `TestFirebaseCoreHostApi` pattern) rather than inventing a new one.

## Dependency added

`firebase_messaging_platform_interface: ^4.8.0` added to `pubspec.yaml`'s `dev_dependencies` (test-only; already a transitive prod dependency via `firebase_messaging`) — needed to reach `MethodChannelFirebaseMessaging.channel` for the mock, and to satisfy the `depend_on_referenced_packages` lint that flagged the otherwise-transitive import.

## Verification

- [x] `fvm flutter analyze` on changed files: clean, no issues
- [x] `fvm flutter test test/core/services/push_notification/notifications_manager_test.dart`: 2/2 pass
- [x] Full suite (`fvm flutter test`): 2291 tests, 1 failure — `test/features/chats/chat_models_test.dart` ("ChatDetailActionEnum should return correct titles"), a pre-existing string-casing mismatch ("Reassign chat" vs "Reassign Chat") unrelated to this change. Confirmed pre-existing by stashing this task's changes and re-running that file alone — same failure occurs on unmodified `develop`. Out of scope for this task; not touched.

## Artifacts

- `lib/core/services/push_notification/notifications_manager.dart` — fix
- `test/core/services/push_notification/notifications_manager_test.dart` — new regression test
- `pubspec.yaml`/`pubspec.lock` — added `firebase_messaging_platform_interface` dev dependency
- Branch pushed to `bla` (per user request — not `origin`): `plan/fix-production-crashes-v180/phase-2/task-03-notification-permission-crash`
- PR creation link (no Bitbucket API token available to open it automatically): https://bitbucket.org/ballastlane/syncro-flutter/pull-requests/new?source=plan/fix-production-crashes-v180/phase-2/task-03-notification-permission-crash&t=1

## Adaptations

None — task.md's plan matched reality exactly; only the test-mocking mechanics (wire-format status codes, dev-dependency addition) required investigation beyond what was written, not a deviation from the task's intent.
