# Status: Home screen unread badge

**Current Status**: complete
**Last Updated**: 2026-05-08
**Agent**: claude-sonnet-4-6
**Branch**: feature/notifications
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-04-16 | not-started | — | Task created |
| 2026-05-08 | complete | claude-sonnet-4-6 | 9 bloc_test tests passing, flutter analyze clean |

## Blockers

None

## Artifacts

- `lib/features/notifications/application/unread_count_cubit.dart`
- `lib/features/notifications/application/unread_count_state.dart`
- `lib/app/dependency/app_repositories.dart` — added NotificationsRepository provider
- `lib/app/dependency/app_providers.dart` — added UnreadCountCubit provider
- `lib/features/home/presentation/home_page.dart` — added AppBar with bell + badge + initial fetch
- `test/features/notifications/application/unread_count_cubit_test.dart`

## Adaptations

- `UnreadCountCubit` subscribes to `AppLifecycleService().onResumedStream` in constructor (avoids extra WidgetsBindingObserver in home screen; stream is already fed by existing `AppStateObserver`)
- Bell icon uses `Badge.count` + `Icons.notifications_outlined` (no bell SVG in assets)
- `onPressed: null` on bell `IconButton` — navigation wired in task-05 (SE-11984/SE-11985)
- 9 tests: fetchUnreadCount success/failure, decrement/increment/reset with edge cases
