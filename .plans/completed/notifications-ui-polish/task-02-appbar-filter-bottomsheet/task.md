# Task: AppBar redesign + filter BottomSheet

**Plan**: notifications-ui-polish
**Task ID**: task-02
**Task Path**: task-02-appbar-filter-bottomsheet
**Depends On**: —
**Branch**: feature/notifications

## Objective

1. Replace the current `TextButton` filter toggle with a filter icon button that opens a BottomSheet.
2. Redesign the AppBar so that "Mark all Read" is an outlined text button and the filter icon sits to its right.

## Design Spec

**AppBar** (left → right):
```
← | Notifications | [Mark all Read] | ≡ (filter icon)
```
- "Mark all Read" only visible when `hasUnread == true` (same as current).
- Filter icon always visible.

**Filter BottomSheet**:
```
┌─────────────────────────┐
│  ▬  (drag handle)       │
│  Select Filter          │
│                         │
│  ✓  Hide Read           │  ← checkmark when filter == unread
│     View All            │  ← no checkmark
│                         │
│  [        Cancel       ] │
└─────────────────────────┘
```
- Tapping an option calls `cubit.setFilter(...)` and closes the sheet.
- "Cancel" closes without changing the filter.
- Use `showModalBottomSheet` — follow the existing pattern in the app (check how other features open BottomSheets).

## Steps

1. **Remove** `_FilterToggleButton` widget entirely.

2. **Add** `_FilterBottomSheet` — a static method or function that shows the modal:

```dart
void _showFilterBottomSheet(BuildContext context, NotificationFilter current) {
  showModalBottomSheet(
    context: context,
    builder: (_) => _FilterSheet(current: current, cubit: context.read<NotificationsCubit>()),
  );
}
```

3. **Redesign** `_NotificationsAppBar.build()`:

```dart
actions: [
  if (hasUnread)
    OutlinedButton(
      onPressed: () => context.read<NotificationsCubit>().markAllAsRead(),
      child: const Text(AppStrings.notificationsMarkAllAsRead),
    ),
  IconButton(
    icon: const Icon(Icons.filter_list),
    onPressed: () => _showFilterBottomSheet(context, state.filter),
  ),
],
```

4. **Add** `_FilterSheet` widget — a stateless widget that renders the BottomSheet content using `ListTile`s with leading `Icon(Icons.check)` (visible only on active filter) and a "Cancel" `ElevatedButton` at the bottom.

5. Update `AppStrings` with any new string constants needed (e.g. `notificationsFilterSelectTitle`, `notificationsFilterHideRead`, `notificationsFilterViewAll`, `notificationsFilterCancel`).

## File Ownership

- `lib/features/notifications/presentation/notifications_page.dart`
- `lib/core/design_system/app_strings.dart`

## Verification

- Tapping filter icon opens the BottomSheet.
- Active filter has a checkmark; inactive does not.
- Selecting an option changes the list and closes the sheet.
- "Cancel" closes without changing the filter.
- "Mark all Read" button is only visible when unread items exist, uses outlined style.
- Run `fvm flutter analyze` — no new issues.
