# Task: Notifications screen UI

**Plan**: notifications-screen
**Phase**: 3 — UI
**Task ID**: task-04
**Task Path**: phase-3/task-04-notifications-screen
**Depends On**: phase-2/task-02-notifications-cubit
**JIRA**: [SE-11982](https://repairtechsolutions.atlassian.net/browse/SE-11982)

## Objective

Build `NotificationsPage` — a paginated list screen with pull-to-refresh, filter toggle (unread/all), swipe-left-to-unread gesture on read items, "Mark all as read" AppBar button, and empty/error states.

## Context

**UX behavior**:
- Default view: unread notifications (filter toggle button to switch to "all")
- Read notifications appear greyed out
- Swipe left on a read notification → toggle back to unread
- "Mark all as read" button in AppBar (visible when at least one item is unread)
- Pull-to-refresh triggers `NotificationsCubit.refresh()`
- Infinite scroll: load more when user reaches last 20% of list
- Empty state: "No notifications" message (tailored to current filter)
- Error state: retry button

**Architecture**:
- Page wires `BlocProvider<NotificationsCubit>` with use cases from `RepositoryProvider`
- Widget listens with `BlocBuilder<NotificationsCubit, NotificationsState>`
- `NotificationItemWidget` is a stateless presentational widget
- Routing: connected to GoRouter via a new named route

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [SE-11982](https://repairtechsolutions.atlassian.net/browse/SE-11982)
- [ ] Verify `phase-2/task-02-notifications-cubit` status is `complete`
- [ ] Grep for an existing paginated list screen for scroll + load-more patterns
- [ ] Verify `status.md` is `not-started`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/notifications/presentation/notifications_page.dart` | create | Main screen |
| `syncro-flutter/lib/features/notifications/presentation/widgets/notification_item_widget.dart` | create | List item widget |
| `syncro-flutter/lib/features/notifications/presentation/widgets/notifications_empty_widget.dart` | create | Empty state |
| `syncro-flutter/lib/features/notifications/presentation/widgets/notifications_error_widget.dart` | create | Error state with retry |
| GoRouter route file (identify path first) | modify | Add `/notifications` route |
| `syncro-flutter/test/features/notifications/presentation/notification_item_widget_test.dart` | create | Widget tests |

### Do NOT Modify

- `syncro-flutter/lib/features/notifications/domain/` — owned by task-01
- `syncro-flutter/lib/features/notifications/infrastructure/` — owned by task-01
- `syncro-flutter/lib/features/notifications/application/notifications_cubit.dart` — owned by task-02
- `syncro-flutter/lib/core/routing/route_cubit.dart` — owned by task-05

## Implementation Steps

### Step 1: Create `NotificationItemWidget`

```dart
class NotificationItemWidget extends StatelessWidget {
  final NotificationItem item;
  final VoidCallback onTap;
  final VoidCallback onToggleUnread; // swipe action on read item

  Widget build(BuildContext context) {
    return Opacity(
      opacity: item.isRead ? 0.5 : 1.0,
      child: ListTile(
        title: Text(item.title),
        subtitle: Text(item.description),
        trailing: Text(_relativeTime(item.createdAt)),
        onTap: onTap,
      ),
    );
  }
}
```

Apply grey opacity when `isRead == true`. Use `Dismissible` or `GestureDetector` for swipe-left-to-unread (only active when `isRead == true`).

### Step 2: Create `NotificationsPage`

- `BlocBuilder` on `NotificationsState`
- `ListView.builder` with `ScrollController` for infinite scroll
- `RefreshIndicator` wrapping the list for pull-to-refresh
- `AppBar` with filter toggle button and "Mark all as read" action
- Loading indicator at bottom of list when `isLoadingMore == true`
- `hasReachedEnd` stops triggering `loadMore()`

### Step 3: Infinite scroll

```dart
_scrollController.addListener(() {
  final maxScroll = _scrollController.position.maxScrollExtent;
  final currentScroll = _scrollController.offset;
  if (currentScroll >= maxScroll * 0.8) {
    context.read<NotificationsCubit>().loadMore();
  }
});
```

### Step 4: Add GoRouter route

Add a `/notifications` named route pointing to `NotificationsPage`. Follow existing route definition patterns. The route should provide `NotificationsCubit` via `BlocProvider` above the page.

## Testing

- [ ] `NotificationItemWidget` renders title and description
- [ ] `NotificationItemWidget` applies opacity when `isRead == true`
- [ ] `NotificationsPage` shows `NotificationsLoading` spinner
- [ ] `NotificationsPage` shows items list in `NotificationsLoaded` state
- [ ] `NotificationsPage` shows empty widget in `NotificationsEmpty` state
- [ ] `NotificationsPage` shows error widget with retry in `NotificationsError` state
- [ ] `flutter test test/features/notifications/presentation/` passes
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required

## Completion Criteria

- [ ] `NotificationsPage` with paginated list, pull-to-refresh, filter toggle, mark-all button
- [ ] `NotificationItemWidget` with read/unread opacity + swipe gesture
- [ ] Empty state and error state widgets
- [ ] GoRouter `/notifications` route registered
- [ ] Widget tests passing
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
- [ ] Changes committed to `plan/notifications-screen/phase-3/task-04-notifications-screen` branch
- [ ] Status updated in `status.md`
