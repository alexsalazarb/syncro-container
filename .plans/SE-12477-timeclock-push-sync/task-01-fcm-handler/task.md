# Task 01 — FCM Event Detection + Background Handler Setup

## JIRA

[SE-12477](https://syncrotech.atlassian.net/browse/SE-12477)

## Objective

Add two detection points for the `timelog_state_changed` silent push:
1. **Foreground**: detect in `NotificationManager.onMessage`, broadcast via a new static stream
2. **Background/terminated**: register a top-level `onBackgroundMessage` handler that writes a
   SharedPreferences flag for later consumption

## Dependencies

None — this is task-01.

## File Ownership

**Create or modify:**
- `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart`
- `syncro-flutter/lib/main.dart`

**Do NOT modify (owned by task-02):**
- `syncro-flutter/lib/features/time_clock/application/latest_time_clock_cubit.dart`

---

## Implementation Steps

### Step 1 — Add `timelogStateChangedStream` to NotificationManager

In `notifications_manager.dart`, add a static broadcast stream alongside the existing
`notificationStream`:

```dart
static final StreamController<void> timelogStateChangedStream =
    StreamController<void>.broadcast();
```

### Step 2 — Detect event in foreground handler

In `_registerEvents()`, inside the `onMessage.listen` callback, after the call to
`_showNotification(message)`, add the event check:

```dart
FirebaseMessaging.onMessage.listen((message) {
  _showNotification(message);
  notificationStream.add(PushNotification.fromRemoteMessage(message));
  // Silent push: timelog updated from web portal
  if (message.data['event'] == 'timelog_state_changed') {
    timelogStateChangedStream.add(null);
  }
});
```

Note: `_showNotification` checks `message.notification != null` before showing a visible
notification, so data-only messages pass through silently without any user-visible alert.

### Step 3 — Add top-level background handler in main.dart

Add the handler function at the top of `main.dart` (before `main()`). It must be a
**top-level function** annotated with `@pragma('vm:entry-point')`:

```dart
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  if (message.data['event'] == 'timelog_state_changed') {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool('timelog_state_refresh_pending', true);
  }
}
```

Add `import 'package:firebase_messaging/firebase_messaging.dart';` if not already present.
Add `import 'package:shared_preferences/shared_preferences.dart';` if not already present.

### Step 4 — Register the background handler

`FirebaseMessaging.onBackgroundMessage` MUST be called before `WidgetsFlutterBinding.ensureInitialized()`.
Place it at the very start of `_initializeApp()`, or in `main()` before `runZonedGuarded`:

```dart
Future<void> _initializeApp() async {
  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler); // ← ADD FIRST
  final widgetsBinding = WidgetsFlutterBinding.ensureInitialized();
  // ... rest unchanged
```

---

## Acceptance Criteria

- `NotificationManager.timelogStateChangedStream` exists and is a broadcast stream
- When `onMessage` fires with `data['event'] == 'timelog_state_changed'`, the stream emits `null`
- When `onMessage` fires with any other event, the stream does NOT emit
- `_firebaseMessagingBackgroundHandler` is registered via `FirebaseMessaging.onBackgroundMessage`
- When called with `event: timelog_state_changed`, it sets `timelog_state_refresh_pending = true` in SharedPreferences
- No visible notification is shown for this event (data-only — no `notification` key)
- `fvm flutter analyze` passes with no new warnings

## Testing

No unit tests for this task — `NotificationManager` is a singleton that wraps static Firebase APIs
and is not easily unit-tested without a full Firebase test environment. Verification is done via
integration / manual QA in task-02's test plan.

## Relevant KB

- `docs/kb-projects/syncro-flutter/technical/integrations/firebase.md`
- `docs/kb-projects/syncro-flutter/product/features/push-notifications.md`
