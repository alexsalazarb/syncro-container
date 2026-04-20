# Task: onTap routing + auto-mark-read

**Plan**: notifications-screen
**Phase**: 4 — Routing
**Task ID**: task-05
**Task Path**: phase-4/task-05-tap-routing
**Depends On**: phase-2/task-02-notifications-cubit, phase-3/task-04-notifications-screen
**JIRA**: [SE-11984](https://repairtechsolutions.atlassian.net/browse/SE-11984)

## Objective

Wire up notification tap: convert `NotificationItem` → `PushNotification` to reuse existing `redirectOnNotification()`, trigger auto-mark-as-read via `PATCH /notifications/:id/read`, and validate all 5 `object_type` routing paths work correctly.

## Context

**Existing infrastructure (DO NOT change the core logic)**:
- `PushNotification` model in `syncro-flutter/lib/core/services/push_notification/push_notification.dart`
  - `static const kSource = "object_type"` (maps to `object_type`)
  - `static const kObjectId = "object_id"` (maps to `object_id`)
  - `static const kUrl = "object_link"` (maps to `object_link`)
- `redirectOnNotification(PushNotification)` in `syncro-flutter/lib/core/routing/route_cubit.dart`
  - Reads `notification.source` (`NotificationSource` enum) via `_parseNotificationSource()`
  - Already handles all 5 object types: `ticket`, `appointment`, `comment`, `rmmalert`, `openstruct`
  - `object_id` arrives as int from BE; mobile calls `.toString()` internally in `PushNotification`

**Conversion approach**:
Convert `NotificationItem` → `PushNotification` using the field mapping above, then call `redirectOnNotification()`. This avoids duplicating routing logic.

```dart
PushNotification _toPushNotification(NotificationItem item) {
  return PushNotification({
    PushNotification.kSource: item.objectType,
    PushNotification.kObjectId: item.objectId.toString(),
    PushNotification.kUrl: item.objectLink,
  });
}
```

**`openStruct` routing note**: `_handleChatNotification()` in `chat_websocket_service.dart` actively initializes the WebSocket and requests interactions — it does NOT require pre-loaded state. The tap flow works independently.

**Auto-mark-read**: call `context.read<NotificationsCubit>().toggleRead(item.id)` only if `!item.isRead` (optimistic update in cubit handles state; use case fires the API call).

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [SE-11984](https://repairtechsolutions.atlassian.net/browse/SE-11984)
- [ ] Verify `phase-2/task-02-notifications-cubit` and `phase-3/task-04-notifications-screen` are both `complete`
- [ ] Read `push_notification.dart` to confirm field constants
- [ ] Read `route_cubit.dart` lines 68–120 (`NotificationSource` enum + `redirectOnNotification`)
- [ ] Verify `status.md` is `not-started`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/notifications/presentation/notifications_page.dart` | modify | Wire `onTap` callback |
| `syncro-flutter/lib/features/notifications/presentation/widgets/notification_item_widget.dart` | modify | Pass `onTap` to `ListTile` |
| `syncro-flutter/test/features/notifications/presentation/notifications_routing_test.dart` | create | Routing validation tests |

### Do NOT Modify

- `syncro-flutter/lib/core/routing/route_cubit.dart` — read-only; contains `redirectOnNotification()`
- `syncro-flutter/lib/core/services/push_notification/push_notification.dart` — read-only
- `syncro-flutter/lib/core/services/chat_websocket_service.dart` — read-only
- `syncro-flutter/lib/features/notifications/application/notifications_cubit.dart` — owned by task-02

## Implementation Steps

### Step 1: Implement `_toPushNotification` conversion

In `notifications_page.dart`, create a private helper:

```dart
PushNotification _toPushNotification(NotificationItem item) {
  return PushNotification({
    PushNotification.kSource: item.objectType,
    PushNotification.kObjectId: item.objectId.toString(),
    PushNotification.kUrl: item.objectLink,
  });
}
```

### Step 2: Wire `onTap` in `NotificationsPage`

In the `ListView.builder` item callback:

```dart
onTap: () {
  // Auto-mark as read (optimistic — cubit handles API call)
  if (!item.isRead) {
    context.read<NotificationsCubit>().toggleRead(item.id);
  }
  // Navigate via existing routing infrastructure
  context.read<RouteCubit>().redirectOnNotification(_toPushNotification(item));
},
```

### Step 3: Validate all 5 object_type paths

Manually verify (or write tests) that each `object_type` routes correctly:

| `object_type` | Expected navigation |
|---------------|---------------------|
| `ticket` | Ticket detail screen |
| `appointment` | Appointment detail screen |
| `comment` | Comment/ticket context screen |
| `rmmalert` | RMM alert screen |
| `openstruct` | Chat interaction screen (via `_handleChatNotification`) |

Use the existing `_parseNotificationSource()` string values exactly — case-insensitive matching is already implemented.

### Step 4: Handle `unknown` object_type gracefully

If `object_type` is unrecognized, `redirectOnNotification()` already handles the `unknown` case — confirm it doesn't crash and logs appropriately.

## Testing

- [ ] `_toPushNotification` maps all 3 fields correctly (`objectType`, `objectId.toString()`, `objectLink`)
- [ ] `onTap` calls `toggleRead(id)` only when `!item.isRead`
- [ ] `onTap` calls `redirectOnNotification` with correct `PushNotification`
- [ ] All 5 `object_type` values produce a non-null route (unit-testable via `RouteCubit`)
- [ ] Unknown `object_type` does not crash
- [ ] `flutter test test/features/notifications/` passes
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] Document `NotificationItem → PushNotification` conversion pattern in KB if not already covered

## Completion Criteria

- [ ] `_toPushNotification()` converts all 3 fields correctly
- [ ] `onTap` triggers `toggleRead` (if unread) + `redirectOnNotification`
- [ ] All 5 `object_type` routing paths validated
- [ ] Unknown `object_type` handled gracefully (no crash)
- [ ] Tests passing
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
- [ ] Changes committed to `plan/notifications-screen/phase-4/task-05-tap-routing` branch
- [ ] Status updated in `status.md`
