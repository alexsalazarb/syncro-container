# Task: Notifications data layer

**Plan**: notifications-screen
**Phase**: 1 — Foundation
**Task ID**: task-01
**Task Path**: phase-1/task-01-data-layer
**Depends On**: None
**JIRA**: [SE-11980](https://repairtechsolutions.atlassian.net/browse/SE-11980)

## Objective

Build the complete data layer for the Notifications feature: `NotificationItem` domain model, repository interface + implementation, deserializers, and all 5 use cases. This is the foundation all other tasks depend on.

## Context

**BE dependency**: SE-11976 (read endpoints) and SE-11977 (write endpoints) must be ready for real API calls. During development, implement the repository with a mock/stub if needed.

**Payload contract**:
```json
{
  "id": 123,
  "title": "string",
  "description": "string",
  "is_read": false,
  "created_at": "2026-04-16T12:00:00Z",
  "object_type": "ticket",
  "object_id": 456,
  "object_link": "https://..."
}
```

**Project conventions** (mandatory):
- Feature directory: `syncro-flutter/lib/features/notifications/`
- Layers: `domain/` → `infrastructure/` → `application/` → `presentation/`
- Use cases live in `infrastructure/` (NOT `domain/`) per project convention
- Repository interface in `domain/`, implementation in `infrastructure/`
- All repo calls return `Either<Failure, T>` — no throws
- Feature repos via `RepositoryProvider` in widget tree, NOT GetIt singletons

**API endpoints**:
- `GET /notifications?page={n}&per_page=25&filter=unread|all`
- `GET /notifications/unread_count`
- `PATCH /notifications/:id/read`
- `PATCH /notifications/:id/unread`
- `PATCH /notifications/mark_all_read`

Look at an existing feature for reference patterns (e.g., assets or tickets feature).

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [SE-11980](https://repairtechsolutions.atlassian.net/browse/SE-11980) — check description AND comments for latest requirements
- [ ] Verify `status.md` is `not-started`
- [ ] Look at an existing feature's domain + infrastructure layer for naming/structure patterns
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/notifications/domain/notification_item.dart` | create | Domain model with `fromJson` |
| `syncro-flutter/lib/features/notifications/domain/notifications_repository.dart` | create | Abstract interface |
| `syncro-flutter/lib/features/notifications/infrastructure/notifications_repository_impl.dart` | create | Real API calls |
| `syncro-flutter/lib/features/notifications/infrastructure/get_notifications_deserializer.dart` | create | List response deserializer |
| `syncro-flutter/lib/features/notifications/infrastructure/get_notifications_use_case.dart` | create | `Either<Failure, List<NotificationItem>>` |
| `syncro-flutter/lib/features/notifications/infrastructure/get_unread_count_use_case.dart` | create | `Either<Failure, int>` |
| `syncro-flutter/lib/features/notifications/infrastructure/mark_as_read_use_case.dart` | create | `Either<Failure, void>` |
| `syncro-flutter/lib/features/notifications/infrastructure/mark_as_unread_use_case.dart` | create | `Either<Failure, void>` |
| `syncro-flutter/lib/features/notifications/infrastructure/mark_all_as_read_use_case.dart` | create | `Either<Failure, void>` |
| `syncro-flutter/test/features/notifications/domain/notification_item_test.dart` | create | Model parsing tests |
| `syncro-flutter/test/features/notifications/infrastructure/get_notifications_deserializer_test.dart` | create | Deserializer tests |
| `syncro-flutter/test/features/notifications/infrastructure/get_notifications_use_case_test.dart` | create | Use case tests |

### Do NOT Modify

- `syncro-flutter/lib/core/routing/route_cubit.dart` — owned by task-05
- `syncro-flutter/lib/core/services/push_notification/push_notification.dart` — owned by task-05

## Implementation Steps

### Step 1: Create `NotificationItem` domain model

```dart
// domain/notification_item.dart
class NotificationItem {
  final int id;
  final String title;
  final String description;
  final bool isRead;
  final DateTime createdAt;
  final String objectType;
  final int objectId;
  final String objectLink;

  const NotificationItem({...});

  factory NotificationItem.fromJson(Map<String, dynamic> json) {
    return NotificationItem(
      id: json['id'] as int,
      title: (json['title'] as String?) ?? '',
      description: (json['description'] as String?) ?? '',
      isRead: json['is_read'] as bool? ?? false,
      createdAt: DateTime.parse(json['created_at'] as String),
      objectType: (json['object_type'] as String?) ?? '',
      objectId: json['object_id'] as int,
      objectLink: (json['object_link'] as String?) ?? '',
    );
  }

  NotificationItem copyWith({bool? isRead}) => NotificationItem(
    id: id,
    title: title,
    description: description,
    isRead: isRead ?? this.isRead,
    createdAt: createdAt,
    objectType: objectType,
    objectId: objectId,
    objectLink: objectLink,
  );
}
```

Use null-safe coercion (`as String?) ?? ''`) for all string fields — same pattern as the crash fixes in `fix-production-crashes-v140`.

### Step 2: Create `NotificationsRepository` interface

```dart
// domain/notifications_repository.dart
abstract class NotificationsRepository {
  Future<Either<Failure, List<NotificationItem>>> getNotifications({
    required int page,
    required int perPage,
    required String filter, // 'unread' | 'all'
  });
  Future<Either<Failure, int>> getUnreadCount();
  Future<Either<Failure, void>> markAsRead(int id);
  Future<Either<Failure, void>> markAsUnread(int id);
  Future<Either<Failure, void>> markAllAsRead();
}
```

### Step 3: Create deserializer

```dart
// infrastructure/get_notifications_deserializer.dart
class GetNotificationsDeserializer {
  static List<NotificationItem> fromJson(Map<String, dynamic> json) {
    final data = json['data'] as List<dynamic>? ?? [];
    return data
        .whereType<Map<String, dynamic>>()
        .map(NotificationItem.fromJson)
        .toList();
  }
}
```

Check the actual BE response envelope shape (may be `{ data: [...] }` or just `[...]`) — adapt accordingly.

### Step 4: Implement `NotificationsRepositoryImpl`

Wire up all 5 endpoints. Follow the existing service call patterns used in `assets` or `tickets` features. Return `Right(result)` on success, `Left(Failure(...))` on error.

### Step 5: Create 5 use cases

Each use case takes the repository as a dependency (injected via constructor). Follow the naming and structure of existing use cases in the project.

## Testing

- [ ] `NotificationItem.fromJson` parses all fields correctly
- [ ] `NotificationItem.fromJson` handles null `title`, `description`, `object_link` → defaults to `''`
- [ ] `fromJson` handles null `is_read` → defaults to `false`
- [ ] Deserializer handles empty array `[]` without crash
- [ ] Deserializer handles null `data` key gracefully
- [ ] `GetNotificationsUseCase` returns `Right(list)` on success
- [ ] `GetNotificationsUseCase` returns `Left(Failure)` on network error
- [ ] `flutter test test/features/notifications/` passes
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required — standard Clean Architecture pattern, already documented at framework level

## Completion Criteria

- [ ] `NotificationItem` domain model created with null-safe `fromJson`
- [ ] `NotificationsRepository` abstract interface defined
- [ ] `NotificationsRepositoryImpl` wired to all 5 endpoints
- [ ] All 5 use cases implemented with `Either<Failure, T>`
- [ ] Test file with model + deserializer + use case coverage
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
- [ ] Changes committed to `plan/notifications-screen/phase-1/task-01-data-layer` branch
- [ ] Status updated in `status.md`
