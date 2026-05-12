# Task: Sync unread badge on every dashboard visit

**Plan**: notifications-ui-polish
**Task ID**: task-01
**Task Path**: task-01-badge-refresh
**Depends On**: —
**Branch**: feature/notifications

## Objective

The unread count badge shown in the dashboard AppBar must reflect the real server count every time the dashboard becomes visible — whether the user returns from Notifications, Settings, any other sub-route, or switches back to the Home tab.

## Context

`DashboardPage` lives inside a `StatefulShellBranch` (GoRouter `StatefulShellRoute.indexedStack`). The widget is **never disposed** — `initState()` runs only once. Currently `fetchUnreadCount()` is called:
- Once in `DashboardPage.initState()` (first load)
- On app resume via `AppLifecycleService` inside `UnreadCountCubit`

Missing: refresh when the user navigates back to `/home` from any sub-route pushed on top (Notifications, Settings, etc.) or switches back to the Home tab.

There is no existing pattern in the app for "refresh on every dashboard visit". `TimeClockButton` uses a one-time init guard (`LatestTimeClockInitial`), not a per-visit refresh. The GoRouter listener is the correct approach.

## Strategy

Listen to `GoRouter.routeInformationProvider` changes inside `_DashboardPageState`. Every time the active route path becomes `AppRoute.home.path` (i.e. `/home`), call `fetchUnreadCount()`. This covers all entry points without coupling any other screen to `UnreadCountCubit`.

## Steps

1. In `_DashboardPageState`, add a `GoRouter` field and a listener method:

```dart
GoRouter? _router;

void _onRouteChanged() {
  final path = _router?.routeInformationProvider.value.uri.path;
  if (path == AppRoute.home.path && mounted) {
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) context.read<UnreadCountCubit>().fetchUnreadCount();
    });
  }
}
```

2. Register and unregister the listener in `didChangeDependencies` and `dispose`:

```dart
@override
void didChangeDependencies() {
  super.didChangeDependencies();
  final router = GoRouter.of(context);
  if (_router != router) {
    _router?.routeInformationProvider.removeListener(_onRouteChanged);
    _router = router;
    _router!.routeInformationProvider.addListener(_onRouteChanged);
  }
}

@override
void dispose() {
  _router?.routeInformationProvider.removeListener(_onRouteChanged);
  super.dispose();
}
```

3. **Remove** the existing `WidgetsBinding.instance.addPostFrameCallback` call from `initState` — it's now redundant since `didChangeDependencies` fires on first mount and also triggers `_onRouteChanged` (the path will be `/home` on first load).

> **Note**: `didChangeDependencies` fires on first mount and whenever an `InheritedWidget` dependency changes. The `GoRouter.of(context)` call reads from `InheritedWidget`, so this is safe.

4. Add the missing imports: `go_router/go_router.dart`, `UnreadCountCubit`.

5. No changes to `NotificationsCubit`, `UnreadCountCubit`, or any other screen.

## File Ownership

- `lib/features/dashboard/presentation/dashboard_page.dart`

## Verification

- Return from Notifications after marking items as read → badge updates.
- Return from Settings → badge updates.
- Switch away to Appointments tab, then switch back to Home → badge updates.
- App resume from background still updates badge (existing `AppLifecycleService` behavior unchanged).
- Run `fvm flutter analyze` — no new issues.
