# Task: iOS — guard `NotificationManager.requestPermissions` against uncaught `firebase_messaging/unknown`

**Plan**: 1.7.1 Crashlytics Fixes (1.7.1-crashlytics)
**Phase**: 2
**Task ID**: task-03
**Task Path**: `phase-2/task-03-notification-permission-crash`
**Depends On**: None
**JIRA**: N/A — create one if desired
**Crashlytics Issues**:
- `307055b68251cef77c20c6ca27b33816` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/307055b68251cef77c20c6ca27b33816)) — firstSeen/lastSeen 1.7.0
- `4d17e7a500b300c48a84f037b2ded77a` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/4d17e7a500b300c48a84f037b2ded77a)) — SIGNAL_FRESH, first appeared 2026-08-21, firstSeen/lastSeen 1.6.0

## Objective

Both issues share the identical error message — `FlutterError: [firebase_messaging/unknown] Notifications are not allowed for this application. Error thrown Service initialization failed.` — and both are FATAL. One stack trace (`4d17e7a5...`) points directly at `NotificationManager.requestPermissions`; the other (`307055b6...`) has an unreadable Dart isolate-snapshot symbol but the same message, strongly suggesting the same call site. Fix both by making `requestPermissions()` resilient to this platform exception instead of letting it crash the app.

## Context

Verified in `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` during planning:

```dart
Future<bool> requestPermissions() async {
  const bool isPatrolTest = bool.fromEnvironment('PATROL', defaultValue: false);
  if (isPatrolTest) return true;
  FirebaseMessaging messaging = FirebaseMessaging.instance;
  NotificationSettings settings = await messaging.requestPermission(
    alert: true,
    badge: true,
    sound: true,
  );
  return settings.authorizationStatus == AuthorizationStatus.authorized;
}
```

There is **no try/catch** around `messaging.requestPermission(...)`. The `[firebase_messaging/unknown] Notifications are not allowed for this application` error is a known `firebase_messaging` iOS platform exception — it surfaces when APNs registration hasn't completed or the OS denies the request in a way the plugin can't cleanly report, and it propagates as an uncaught `FlutterError` since nothing here catches it. `requestPermissions()` is called from `registerToTokenChanges()` (line ~195), which is itself called from `register()` — part of the app's init pipeline (per `AGENTS.md`: `main.dart` → Firebase → Hive → GetIt → Notifications). An uncaught exception this early in startup is consistent with both issues being FATAL.

Every other `try`/`catch` in this same file (`_subscribeToAuthStateChanges`, `logout`, `clearFCMToken`, `registerToTokenChanges`'s own `messaging.getToken()` call) already wraps the risky call and logs via `logger()` on failure — `requestPermissions()` is the one exception (no pun intended) to that pattern in this file.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase bla develop`
- [ ] Create the task branch: `git switch -c plan/1.7.1-crashlytics/phase-2/task-03-notification-permission-crash`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` | modify | Wrap `messaging.requestPermission(...)` in `requestPermissions()` in try/catch, matching the existing `logger()`-on-failure pattern used elsewhere in this file; return `false` on failure instead of letting the exception propagate |
| `syncro-flutter/test/core/services/push_notification/notifications_manager_test.dart` | create | No test file exists for this class yet (confirmed via `fd` during planning) |

### Do NOT Modify

- `registerToTokenChanges()`'s existing `messaging.getToken()` try/catch — already correct, out of scope
- Any file owned by another task in this plan

## Implementation Steps

### Step 1: Add error handling
In `requestPermissions()`, wrap the `messaging.requestPermission(...)` call in try/catch. On catch: `logger('Error requesting notification permissions: $e')`, return `false` (mirrors the existing `false`-on-denial return path so callers don't need new handling).

### Step 2: Confirm callers degrade gracefully
Check `registerToTokenChanges()` (the only caller): it already does `if (!permissionGranted) return false;` — confirm this remains a graceful no-crash path with the new catch in place (it should, by construction, but verify).

## Testing

- [ ] New test: `requestPermissions()` returns `false` (not an unhandled exception) when `FirebaseMessaging.instance.requestPermission()` throws a `FirebaseException`/`PlatformException` with code `firebase_messaging/unknown` — mock via the existing `NotificationManager.overrideInstance()` test seam, or mock the `FirebaseMessaging` platform interface per this project's existing Firebase test conventions (check `docs/kb-projects/syncro-flutter/technical/testing/feature-flag-manager-testability.md` for the project's general approach to Firebase-backed test doubles — it documents a similar "No Firebase App '[DEFAULT]'" pitfall worth avoiding here too).
- [ ] Existing behavior preserved: `requestPermissions()` still returns `true`/`false` correctly on the non-throwing authorized/denied paths.
- [ ] `fvm flutter analyze` passes
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB update required — this is a straightforward defensive-catch fix consistent with the file's existing pattern, not a new pattern worth documenting.

## Completion Criteria

- [ ] `requestPermissions()` no longer propagates an uncaught exception on `firebase_messaging/unknown`
- [ ] New regression test passes
- [ ] No new occurrence of either issue in Crashlytics after the fix ships (follow-up check, not blocking this task's completion)
