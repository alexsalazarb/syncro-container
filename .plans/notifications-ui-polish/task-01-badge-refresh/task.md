# Task: Sync unread badge on return to dashboard

**Plan**: notifications-ui-polish
**Task ID**: task-01
**Task Path**: task-01-badge-refresh
**Depends On**: —
**Branch**: feature/notifications

## Objective

The unread count badge shown in the dashboard AppBar must reflect the real server count after the user returns from the Notifications screen — regardless of how many items they marked as read (one, several, or all).

## Context

`UnreadCountCubit` (provided at the root `MultiRepositoryProvider` level) already exposes `fetchUnreadCount()`. The Notifications screen creates its own `NotificationsCubit` via `BlocProvider` and has no reference to `UnreadCountCubit`.

The chosen strategy is **refresh on pop** via `PopScope`:
- No coupling between `NotificationsCubit` and `UnreadCountCubit`.
- A single API call when the user navigates back — always reflects real server state.
- Avoids local counter drift (e.g. if the user marks 3 as read then navigates back via deep-link).

## Steps

1. Wrap the `AppScaffold` inside `NotificationsPage.build()` with a `PopScope`:

```dart
return PopScope(
  canPop: true,
  onPopInvokedWithResult: (didPop, _) {
    if (didPop) {
      context.read<UnreadCountCubit>().fetchUnreadCount();
    }
  },
  child: BlocProvider(
    create: ...,
    child: const AppScaffold(...),
  ),
);
```

> **Important**: `context.read<UnreadCountCubit>()` must be called on the outer `context` (the one that has `UnreadCountCubit` in scope), not inside the inner `BlocProvider`.

2. Add the missing import for `UnreadCountCubit`.

3. No changes to `NotificationsCubit` or `UnreadCountCubit` — they stay decoupled.

## File Ownership

- `lib/features/notifications/presentation/notifications_page.dart`

## Verification

- Navigate to Notifications, mark 2 items as read, press back → badge count decreases.
- Navigate to Notifications, tap "Mark all Read", press back → badge shows 0 (or disappears).
- Run `fvm flutter analyze` — no new issues.
