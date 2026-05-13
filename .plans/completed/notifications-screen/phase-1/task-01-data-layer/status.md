# Status: Notifications data layer

**Current Status**: complete
**Last Updated**: 2026-05-12
**Agent**: claude-sonnet-4-6
**Branch**: feature/notifications
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-04-16 | not-started | — | Task created |
| 2026-05-06 | complete | claude-sonnet-4-6 | All files implemented, 31 tests passing, flutter analyze clean |
| 2026-05-12 | complete | claude-sonnet-4-6 | Bug fix: 3 deserialization bugs found during QA |

## Blockers

None

## Artifacts

- `lib/features/notifications/domain/notification_item.dart`
- `lib/features/notifications/domain/notifications_repository.dart`
- `lib/features/notifications/domain/notifications_list_response.dart`
- `lib/features/notifications/infrastructure/notification_item_deserializer.dart`
- `lib/features/notifications/infrastructure/notifications_list_deserializer.dart`
- `lib/features/notifications/infrastructure/unread_count_deserializer.dart`
- `lib/features/notifications/infrastructure/mark_all_as_read_deserializer.dart`
- `lib/features/notifications/infrastructure/notifications_repository_impl.dart`
- `lib/features/notifications/infrastructure/get_notifications_use_case.dart`
- `lib/features/notifications/infrastructure/get_unread_count_use_case.dart`
- `lib/features/notifications/infrastructure/mark_as_read_use_case.dart`
- `lib/features/notifications/infrastructure/mark_as_unread_use_case.dart`
- `lib/features/notifications/infrastructure/mark_all_as_read_use_case.dart`
- `lib/core/networking/commons/app_requests_notifications.dart`
- `test/features/notifications/domain/notification_item_test.dart`
- `test/features/notifications/infrastructure/notifications_list_deserializer_test.dart`
- `test/features/notifications/infrastructure/use_cases_test.dart`

## Adaptations

- `objectLink` implementado como `String?` (nullable) según el ticket SE-11980, no como `String` como indicaba el task.md
- `markAllAsRead()` devuelve `Either<Failure, int>` (updated_count) según el ticket, no `Either<Failure, void>`
- Tests usan stubs manuales (no mockito @GenerateMocks) para evitar regeneración de código
- **Bug fix 2026-05-12**: `json['object_id'] as int` crasheaba con `null` en el payload real → corregido a `(json['object_id'] as int?) ?? 0`
- **Bug fix 2026-05-12**: `data['meta']` no existía en el response del BE — la key real es `data['metadata']`
- **Bug fix 2026-05-12**: `meta['page']` no existía — la key real es `meta['current_page']` (alineado con el payload del BE)
