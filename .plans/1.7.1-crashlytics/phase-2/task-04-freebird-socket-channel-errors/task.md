# Task: guard `FirebirdSocketService` channel join/push against phoenix_socket timeout/assertion errors

**Plan**: 1.7.1 Crashlytics Fixes (1.7.1-crashlytics)
**Phase**: 2
**Task ID**: task-04
**Task Path**: `phase-2/task-04-freebird-socket-channel-errors`
**Depends On**: None
**JIRA**: N/A — create one if desired
**Crashlytics Issues**:
- `aa0323e5c16c3e7546f84452e28eb8a3` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/aa0323e5c16c3e7546f84452e28eb8a3)) — `package:phoenix_socket/src/push.dart - Push.future`, `Instance of 'ChannelTimeoutException'`, NON_FATAL
- `be06f8e7c72127f022185820c3d9ad4b` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/be06f8e7c72127f022185820c3d9ad4b)) — `package:phoenix_socket/src/channel.dart - PhoenixChannel.join`, `Failed assertion: line 223 pos 12: '!_joinedOnce': is not true.`, NON_FATAL, lastSeen **1.7.1**

## Objective

Guard `FirebirdSocketService`'s channel join/push against unhandled `phoenix_socket` errors: a channel-join timeout and a duplicate-join assertion. Both stack traces point inside the `phoenix_socket` package itself — meaning app code called into it without the guards the package expects (a timeout wrapper, and a check that the channel hasn't already been joined).

## Context

Verified during planning: this app has **two** `phoenix_socket` consumers, with very different levels of defensiveness:

- `lib/core/services/chat_websocket_service.dart` (1228 lines, the main chat feature) — **already hardened**: its channel join is wrapped in `.timeout(...)` with a `catchError`-equivalent try/catch (lines ~340-360), and pushes are guarded with `canPush` checks (line 765) before calling `.push(...)`. **Out of scope — do not modify.**
- `lib/core/services/freebird_websocket_service.dart` (200 lines, the Kabuto asset-live thumbnail socket used on Asset Detail) — **not hardened**. `init()` does:

```dart
await _channel.join().future.then((value) {
  if (value.isOk) {
    _channel.push('request_thumbnail', {});
  }
  return value;
});
```

No `.timeout(...)`, no `.catchError(...)` on the join future, and no guard against calling `join()` again if `init()` runs a second time on an already-joined channel (the `!_joinedOnce` assertion is exactly that: phoenix_socket's own internal check that `join()` wasn't called twice on the same channel). Given `FirebirdSocketService` is a singleton (`factory FirebirdSocketService() => _singleton`) with a mutable `late PhoenixChannel _channel`, calling `init()` more than once (e.g. re-entering Asset Detail) is a plausible trigger for the double-join assertion.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase bla develop`
- [ ] Create the task branch: `git switch -c plan/1.7.1-crashlytics/phase-2/task-04-freebird-socket-channel-errors`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/services/freebird_websocket_service.dart` | modify | Add timeout + error handling on `_channel.join().future`; guard against re-entrant `init()` calls joining an already-joined channel |
| `syncro-flutter/test/core/services/freebird_websocket_service_test.dart` | create | No test file exists for this class yet (confirmed via `fd` during planning) |

### Do NOT Modify

- `syncro-flutter/lib/core/services/chat_websocket_service.dart` — already hardened, out of scope (see plan overview's Kill Criteria: don't grow this into an infra rewrite of the shared reconnect logic)
- `syncro-flutter/test/core/services/chat_websocket_service_connect_test.dart` — belongs to the file above, out of scope

## Implementation Steps

### Step 1: Guard against double-join
Add an `_isJoined` (or reuse `PhoenixChannel`'s own state if it exposes one — check the `phoenix_socket: ^0.7.6` package API) boolean/check in `init()` before calling `_channel.join()`, mirroring the intent of `chat_websocket_service.dart`'s `_isMessageChannelJoined` flag (line 94) — but do not copy chat's full implementation wholesale; keep this proportional to `FirebirdSocketService`'s much smaller scope.

### Step 2: Add timeout + error handling on join
Wrap `_channel.join().future` with `.timeout(...)` (pick a reasonable duration — mirror `chat_websocket_service.dart`'s 15s if there's no reason to differ) and catch/log the resulting `TimeoutException`/`ChannelTimeoutException` via `logger()`, matching this file's existing catch-and-log style (see `init()`'s outer try/catch, line 73-75).

### Step 3: Guard the push
Only call `_channel.push('request_thumbnail', {})` if the channel is confirmed joined and pushable — check whether `PhoenixChannel` exposes a `canPush`-equivalent (as used in `chat_websocket_service.dart` line 765) in this package version.

## Testing

- [ ] New test: calling `init()` twice on an already-joined channel does not throw the `!_joinedOnce` assertion (mock `PhoenixSocket`/`PhoenixChannel` — check if `chat_websocket_service_connect_test.dart` already has reusable phoenix_socket mocks/fakes to follow the same convention).
- [ ] New test: a join timeout is caught and logged, not left to propagate as an unhandled `ChannelTimeoutException`.
- [ ] Existing behavior preserved: a successful join still triggers `request_thumbnail` push and the existing `assetSocketStream` events still fire correctly.
- [ ] `fvm flutter analyze` passes
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Update `docs/kb-projects/syncro-flutter/technical/integrations/phoenix-socket.md` to note that `FirebirdSocketService` now follows the same join/timeout-guard pattern as `ChatWebSocketService` (if the doc currently only describes the chat service) — run `check-kb-index` if the doc's structure changes materially.

## Completion Criteria

- [ ] `FirebirdSocketService.init()` no longer propagates unhandled `ChannelTimeoutException` or `!_joinedOnce` assertion errors
- [ ] New regression tests pass
- [ ] No new occurrence of either issue in Crashlytics after the fix ships (follow-up check, not blocking this task's completion)
