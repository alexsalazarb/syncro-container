# Task: Item widget layout — 3 lines, time+chevron, swipe to mark-as-read

**Plan**: notifications-ui-polish
**Task ID**: task-03
**Task Path**: task-03-item-widget-redesign
**Depends On**: —
**Branch**: feature/notifications

## Objective

Redesign `NotificationItemWidget` to match the design spec:

1. **Body**: up to 3 lines (currently 2).
2. **Top-right**: `CustomTimeAgoWidget` + `Icon(Icons.chevron_right)` side by side.
3. **Swipe on unread items**: reveals a gray background with ✕ icon + "Mark as Read" label → calls `markAsRead()`.
4. **Current swipe on read items** (mark as unread) is REMOVED — the design does not include it.

## Design Spec

```
┌─────────────────────────────────────────┐
│ Title (bold if unread)       5m ago  >  │
│ Body line 1                             │
│ Body line 2                             │
│ Body line 3 (if needed)                 │
└─────────────────────────────────────────┘
```

On swipe (endToStart) of an **unread** item:
```
┌──────────────────────┬──────────────────┐
│ [item content]       │  ✕  Mark as Read │
└──────────────────────┴──────────────────┘
```
- Background of the revealed action: gray (`Colors.grey.shade700` or equivalent from the theme).
- Icon: `Icons.close` (the ✕).
- Label: "Mark as Read" below the icon.

## Steps

1. **Replace** the `ListTile` layout with a custom `InkWell` + `Padding` + `Column`/`Row` so we control the trailing area precisely:

```
Row(children: [
  Expanded(child: title),
  Row(children: [
    CustomTimeAgoWidget(date: item.createdAt),
    const SizedBox(width: 4),
    Icon(Icons.chevron_right, size: 16),
  ])
])
```

2. **Change** `maxLines` on description from `2` → `3`.

3. **Wrap all items** (read and unread) in `Dismissible` with `direction: DismissDirection.endToStart`:
   - For **unread** items: background = gray + ✕ icon + "Mark as Read" label → `confirmDismiss` calls `onMarkAsRead()`, returns `false`.
   - For **read** items: no swipe (remove the existing "mark as unread" swipe) — OR keep it if the design supports it. Per the design images provided, only "Mark as Read" is shown. Remove "mark as unread" swipe.

4. **Update** `NotificationItemWidget` constructor: add `required VoidCallback onMarkAsRead` parameter, remove `onMarkAsUnread`.

5. **Update** the call site in `notifications_page.dart`: replace `onMarkAsUnread` with `onMarkAsRead` that calls `context.read<NotificationsCubit>().markAsRead(item.id)`.

> **Note**: `markAsRead()` is also called on `onTap` for unread items. After this task, swipe and tap both trigger `markAsRead()` for unread items. That's correct — two entry points, same action.

6. Update `AppStrings` with `notificationsMarkAsRead` if not already present.

## File Ownership

- `lib/features/notifications/presentation/widgets/notification_item_widget.dart`
- `lib/features/notifications/presentation/notifications_page.dart`
- `lib/core/design_system/app_strings.dart`

## Verification

- Unread item shows bold title, 3-line body, time ago + chevron at top right.
- Read item shows normal weight title, 50% opacity.
- Swiping unread item reveals gray background with ✕ + "Mark as Read" — calling the action marks it as read without removing from list.
- Swiping a read item does nothing (no Dismissible).
- Run `fvm flutter analyze` — no new issues.
- Run `fvm flutter test test/features/notifications/` — all tests pass.
