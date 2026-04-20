# Task: NotificationsCubit + state

**Plan**: notifications-screen
**Phase**: 2 — Logic
**Task ID**: task-02
**Task Path**: phase-2/task-02-notifications-cubit
**Depends On**: phase-1/task-01-data-layer
**JIRA**: [SE-11981](https://repairtechsolutions.atlassian.net/browse/SE-11981)

## Objective

Build `NotificationsCubit` with paginated fetching, unread/all filter toggle, optimistic read/unread updates, and mark-all-as-read. This cubit is the state source for both the notifications screen (task-04) and the routing tap handler (task-05).

## Context

**Project conventions** (mandatory):
- Use `safeEmit()` instead of `emit()` in ALL cubits — project-wide constraint
- Layer: `application/` inside the `notifications` feature directory
- Use cases are injected via constructor (received from `RepositoryProvider`)
- `Either<Failure, T>` — unwrap with `.fold()`, never throw from cubit

**UX behavior to implement**:
- Default filter: `unread` — toggle to `all`
- Pagination: `page` starts at 1, `per_page = 25`
- Optimistic update: flip `isRead` in local state immediately, then call API; revert on failure
- `markAllAsRead`: clear or grey out all items in current list optimistically, then confirm with API
- `hasReachedEnd` flag: when API returns fewer items than `per_page`, stop loading more

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [SE-11981](https://repairtechsolutions.atlassian.net/browse/SE-11981)
- [ ] Verify `phase-1/task-01-data-layer` status is `complete`
- [ ] Look at an existing cubit in the project for `safeEmit()` usage and state patterns
- [ ] Verify `status.md` is `not-started`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/notifications/application/notifications_cubit.dart` | create | Cubit |
| `syncro-flutter/lib/features/notifications/application/notifications_state.dart` | create | State classes |
| `syncro-flutter/test/features/notifications/application/notifications_cubit_test.dart` | create | bloc_test tests |

### Do NOT Modify

- `syncro-flutter/lib/features/notifications/domain/` — owned by task-01
- `syncro-flutter/lib/features/notifications/infrastructure/` — owned by task-01
- `syncro-flutter/lib/core/routing/route_cubit.dart` — owned by task-05

## Implementation Steps

### Step 1: Define `NotificationsState`

```dart
// application/notifications_state.dart

enum NotificationsFilter { unread, all }

abstract class NotificationsState {}

class NotificationsInitial extends NotificationsState {}

class NotificationsLoading extends NotificationsState {}

class NotificationsLoaded extends NotificationsState {
  final List<NotificationItem> items;
  final int currentPage;
  final bool hasReachedEnd;
  final NotificationsFilter filter;
  final bool isLoadingMore;

  const NotificationsLoaded({
    required this.items,
    required this.currentPage,
    required this.hasReachedEnd,
    required this.filter,
    this.isLoadingMore = false,
  });

  NotificationsLoaded copyWith({...}) => ...;
}

class NotificationsEmpty extends NotificationsState {
  final NotificationsFilter filter;
}

class NotificationsError extends NotificationsState {
  final String message;
}
```

### Step 2: Implement `NotificationsCubit`

```dart
// application/notifications_cubit.dart
class NotificationsCubit extends Cubit<NotificationsState> {
  NotificationsCubit({
    required GetNotificationsUseCase getNotifications,
    required GetUnreadCountUseCase getUnreadCount,
    required MarkAsReadUseCase markAsRead,
    required MarkAsUnreadUseCase markAsUnread,
    required MarkAllAsReadUseCase markAllAsRead,
  }) : super(NotificationsInitial());

  static const int _perPage = 25;

  Future<void> load({NotificationsFilter filter = NotificationsFilter.unread}) async { ... }

  Future<void> loadMore() async { ... }

  Future<void> toggleFilter() async { ... }

  Future<void> toggleRead(int id) async {
    // 1. Find item, flip isRead in state (optimistic)
    // 2. Call markAsRead or markAsUnread
    // 3. On failure: revert state + show error
  }

  Future<void> markAllRead() async {
    // 1. Mark all items isRead=true in state (optimistic)
    // 2. Call markAllAsRead use case
    // 3. On failure: revert + show error
  }

  Future<void> refresh() async {
    // Reset to page 1 with current filter
  }
}
```

Use `safeEmit()` everywhere instead of `emit()`.

### Step 3: Wire use cases from constructor

The cubit does NOT use GetIt. Use cases are injected via constructor. The presentation layer provides them via `BlocProvider` + `RepositoryProvider`.

## Testing

Use `bloc_test` + `mockito` (existing test setup).

- [ ] `load()` emits `NotificationsLoading` then `NotificationsLoaded` with items
- [ ] `load()` emits `NotificationsEmpty` when API returns empty list
- [ ] `load()` emits `NotificationsError` on failure
- [ ] `loadMore()` appends items to existing list
- [ ] `loadMore()` sets `hasReachedEnd = true` when fewer than `perPage` items returned
- [ ] `toggleRead()` optimistically flips `isRead` in state
- [ ] `toggleRead()` reverts on API failure
- [ ] `markAllRead()` marks all items as read optimistically
- [ ] `toggleFilter()` reloads with new filter from page 1
- [ ] `flutter test test/features/notifications/application/` passes
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required

## Completion Criteria

- [ ] `NotificationsState` with all variants defined
- [ ] `NotificationsCubit` with `load`, `loadMore`, `refresh`, `toggleFilter`, `toggleRead`, `markAllRead`
- [ ] All `safeEmit()` — no bare `emit()` calls
- [ ] `bloc_test` coverage for all public methods
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
- [ ] Changes committed to `plan/notifications-screen/phase-2/task-02-notifications-cubit` branch
- [ ] Status updated in `status.md`
