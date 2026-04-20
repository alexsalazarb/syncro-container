# Task: Home screen unread badge

**Plan**: notifications-screen
**Phase**: 2 — Logic
**Task ID**: task-03
**Task Path**: phase-2/task-03-home-badge
**Depends On**: phase-1/task-01-data-layer
**JIRA**: [SE-11983](https://repairtechsolutions.atlassian.net/browse/SE-11983)

## Objective

Add a bell icon with an unread count badge to the home screen (bottom navigation or AppBar). The badge refreshes on app foreground and updates in response to mark-read/unread actions. Uses `GetUnreadCountUseCase` from task-01.

## Context

**UX behavior**:
- Badge shows unread count only (hidden or `0` when no unread)
- Refreshes when app comes to foreground (`WidgetsBindingObserver.didChangeAppLifecycleState`)
- Updates after any mark-read / mark-unread action from the notifications screen
- Tap on bell icon → navigate to Notifications screen

**Project conventions** (mandatory):
- `safeEmit()` in any cubit
- `Either<Failure, T>` from use case, handle failure gracefully (show 0 or hide badge on error)

Explore the home screen / bottom navigation files to understand the current structure before modifying.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [SE-11983](https://repairtechsolutions.atlassian.net/browse/SE-11983)
- [ ] Verify `phase-1/task-01-data-layer` status is `complete`
- [ ] Grep for the home screen widget and bottom navigation to understand current structure
- [ ] Verify `status.md` is `not-started`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/notifications/application/unread_count_cubit.dart` | create | Thin cubit for badge count |
| `syncro-flutter/lib/features/notifications/application/unread_count_state.dart` | create | State |
| Home screen / navigation widget (identify path first) | modify | Add bell icon + badge + WidgetsBindingObserver |
| `syncro-flutter/test/features/notifications/application/unread_count_cubit_test.dart` | create | Cubit tests |

### Do NOT Modify

- `syncro-flutter/lib/features/notifications/domain/` — owned by task-01
- `syncro-flutter/lib/features/notifications/infrastructure/` — owned by task-01
- `syncro-flutter/lib/features/notifications/application/notifications_cubit.dart` — owned by task-02
- `syncro-flutter/lib/core/routing/route_cubit.dart` — owned by task-05

## Implementation Steps

### Step 1: Create `UnreadCountCubit`

```dart
// application/unread_count_cubit.dart
class UnreadCountCubit extends Cubit<UnreadCountState> {
  UnreadCountCubit({required GetUnreadCountUseCase getUnreadCount})
      : super(const UnreadCountState(count: 0));

  Future<void> refresh() async {
    final result = await _getUnreadCount();
    result.fold(
      (_) => safeEmit(const UnreadCountState(count: 0)), // fail silently
      (count) => safeEmit(UnreadCountState(count: count)),
    );
  }
}
```

### Step 2: Add `WidgetsBindingObserver` to home screen

In the home screen stateful widget:
```dart
@override
void didChangeAppLifecycleState(AppLifecycleState state) {
  if (state == AppLifecycleState.resumed) {
    context.read<UnreadCountCubit>().refresh();
  }
}
```

### Step 3: Add bell icon + badge to navigation

Add a badge widget wrapping a bell icon. Use existing badge patterns in the project if available, or `Stack` + `Positioned` for the counter overlay. Only show the badge when count > 0.

### Step 4: Register `UnreadCountCubit` in widget tree

Provide `UnreadCountCubit` above the home screen via `BlocProvider`, passing `GetUnreadCountUseCase` from `RepositoryProvider`. Do NOT use GetIt.

## Testing

- [ ] `UnreadCountCubit.refresh()` emits correct count on success
- [ ] `UnreadCountCubit.refresh()` emits `count: 0` on failure (silent)
- [ ] Badge not shown when count is 0
- [ ] Badge shows count when count > 0
- [ ] `flutter test test/features/notifications/application/unread_count_cubit_test.dart` passes
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required

## Completion Criteria

- [ ] `UnreadCountCubit` with `refresh()` method
- [ ] Home screen `WidgetsBindingObserver` wired to refresh count on foreground
- [ ] Bell icon with badge in navigation
- [ ] Tap on bell navigates to Notifications screen (route wired when task-04 branch is merged)
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
- [ ] Changes committed to `plan/notifications-screen/phase-2/task-03-home-badge` branch
- [ ] Status updated in `status.md`
