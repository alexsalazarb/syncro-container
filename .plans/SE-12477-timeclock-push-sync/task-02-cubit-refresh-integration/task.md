# Task 02 — LatestTimeClockCubit Refresh Integration

## JIRA

[SE-12477](https://syncrotech.atlassian.net/browse/SE-12477)

## Objective

Wire `LatestTimeClockCubit` to react to both the foreground push stream and the background flag,
triggering a full API refresh when a `timelog_state_changed` push is received. The badge (green dot
on `TimeClockButton`) updates automatically when the cubit emits a new state — no extra badge logic
needed.

## Dependencies

**task-01-fcm-handler** must be complete — this task depends on
`NotificationManager.timelogStateChangedStream` being available.

## File Ownership

**Modify:**
- `syncro-flutter/lib/features/time_clock/application/latest_time_clock_cubit.dart`

**Create:**
- `syncro-flutter/test/features/time_clock/application/latest_time_clock_cubit_test.dart`

**Do NOT modify (owned by task-01):**
- `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart`
- `syncro-flutter/lib/main.dart`

---

## Implementation Steps

### Step 1 — Add stream subscriptions to LatestTimeClockCubit

Add two `StreamSubscription` fields and subscribe in the constructor body:

```dart
import 'dart:async';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:syncro/core/services/app_lifecycle_service.dart';
import 'package:syncro/core/services/push_notification/notifications_manager.dart';

// Inside the class:
StreamSubscription<void>? _pushSubscription;
StreamSubscription<void>? _resumeSubscription;
```

In the constructor, after `super(LatestTimeClockInitial())`:

```dart
_pushSubscription = NotificationManager.timelogStateChangedStream.listen((_) {
  if (!isUpdating) refresh();
});

_resumeSubscription = AppLifecycleService().onResumedStream.listen((_) async {
  final prefs = await SharedPreferences.getInstance();
  if (prefs.getBool('timelog_state_refresh_pending') == true) {
    await prefs.remove('timelog_state_refresh_pending');
    if (!isUpdating) refresh();
  }
});
```

### Step 2 — Cancel subscriptions on close

In the `close()` override:

```dart
@override
Future<void> close() {
  _pushSubscription?.cancel();
  _resumeSubscription?.cancel();
  noteController.dispose();
  return super.close();
}
```

### Step 3 — Run format

```bash
cd syncro-flutter && fvm dart format lib/features/time_clock/application/latest_time_clock_cubit.dart
```

---

## Acceptance Criteria

- When `NotificationManager.timelogStateChangedStream` emits, `LatestTimeClockCubit` calls `refresh()`
- When `isUpdating == true` and the stream emits, refresh is NOT triggered (no interference with in-progress clock action)
- On `AppLifecycleService.onResumedStream` emit: if SP flag is `true` → refresh is called and flag is cleared; if flag is `false` or absent → no refresh
- `TimeClockButton` green dot reflects the updated state after the API refresh completes (automatic via `BlocBuilder` — no additional wiring needed)
- Both subscriptions are cancelled in `close()`
- `fvm flutter test test/features/time_clock/` passes

---

## Testing

New file: `test/features/time_clock/application/latest_time_clock_cubit_test.dart`

Test conventions (match existing test files in the project):
- Stub repo via `_StubRepo implements TimeClockRepository` (no mockito)
- Use `bloc_test` / `blocTest<LatestTimeClockCubit, LatestTimeClockState>()`
- Use `FakeAsync` or `StreamController` to simulate push events

**Required test cases:**

```
group('timelog push refresh') {
  test('refresh() called when timelogStateChangedStream emits and isUpdating is false')
  test('refresh() NOT called when timelogStateChangedStream emits and isUpdating is true')
}

group('background resume refresh') {
  test('refresh() called on resume when timelog_state_refresh_pending is true')
  test('SP flag cleared after refresh triggered on resume')
  test('refresh() NOT called on resume when flag is absent')
  test('refresh() NOT called on resume when isUpdating is true')
}

group('subscriptions') {
  test('push and resume subscriptions are cancelled on close')
}
```

## Relevant KB

- `docs/kb-projects/syncro-flutter/technical/architecture/architecture-patterns.md`
- `docs/kb-projects/syncro-flutter/technical/integrations/firebase.md`
