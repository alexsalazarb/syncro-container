# Task: Defect — ChatWebSocketService double-join race ('!_joinedOnce')

**Plan**: Fix Production Crashes — v1.8.0
**Task ID**: task-09
**Task Path**: phase-2/task-09-defect-chat-websocket-joinedonce
**Depends On**: None
**Blocks**: None
**JIRA**: SE-13805
**Severity**: P2 — NON_FATAL, but this task resolves the exact issue this plan had already logged as "not investigated by anyone yet" (see Corrected Assumption below)
**Found By**: Alex Salazar (via Claude Code session, reported a live Crashlytics/log trace showing the crash reproducing after the v1.8.0 Firebase Stability changes)
**Found During**: Ad-hoc investigation, not a task in this plan — surfaced because the user asked to analyze a fresh production log

## Corrected Assumption (read this first)

This plan's own `overview.md` asserted twice that `chat_websocket_service.dart` was already fully guarded and out of scope:

- **Out of Scope** (line 63): _"`chat_websocket_service.dart` — already has guarded join/timeout/error-handling logic (verified during the original planning); not touched by task-04"_
- **Key Files** (line 116): _"chat_websocket_service.dart | Reference only (already hardened) — out of scope, do not modify"_

That assumption was **incomplete**. `connect()` (the low-level socket transport call) does have a reentrancy guard (`_connectingCompleter`, added in SE-11997) — but the two higher-level methods that actually call `PhoenixChannel.join()` did not:

- `_tryInitialConnection()` (joins the user channel `_channel`)
- `_createAndJoinChannel()` (joins the message channel `_messageChannel`)

This is precisely the issue this same plan flagged, unresolved, under **"Genuinely not yet planned by anyone"** (overview.md line 68): `ChatWebSocketService.startNewChat` "Failed to join channel" (Android `6cca598ad0eed80d3c1a344cb1d3af95`, iOS `f92cc8bc8266594c291fa984b0c75366`). `startNewChat()` calls `_createAndJoinChannel()` directly — same code path, same root cause. This task resolves that open item and corrects the two stale claims above (see Plan Updates at the bottom).

Not a scope violation of task-04: task-04 (`FirebirdSocketService`) never touched this file; the "out of scope" note applied only to task-04's own boundary, not to future work.

## Bug Description

`ChatWebSocketService` crashes with `phoenix_socket`'s `PhoenixChannel.join()` assertion:

```
'package:phoenix_socket/src/channel.dart': Failed assertion: line 223 pos 12: '!_joinedOnce': is not true.
```

Reproduced live via a user-provided log trace (2026-09-08):

```
[phoenix_socket.socket] Socket open
[log] ==>>✅ Socket connected (attempt 1).
[log] ==>>❌ Channel join failed: 'package:phoenix_socket/src/channel.dart': Failed assertion: line 223 pos 12: '!_joinedOnce': is not true.
flutter: The following exception was thrown Unexpected channel join error: ...
  #2 PhoenixChannel.join (package:phoenix_socket/src/channel.dart:223:12)
  #3 ChatWebSocketService._tryInitialConnection (package:syncro/core/services/chat_websocket_service.dart:343:25)
  #4 ChatWebSocketService.getInteractions (package:syncro/core/services/chat_websocket_service.dart:679:7)
[log] ==>>🔌 WebSocket disconnected.
[log] ==>>❌ getInteractions: channel not ready after reinit.
[log] ==>>❌ Channel join failed: TimeoutException: Channel join timeout (15 s)
[log] ==>>🔌 WebSocket disconnected.
[log] ==>>Subscribed to topic syncro-staging1-azure-userID-256
```

Reported as happening **more often** after the v1.8.0 Firebase Stability changes (this plan) — likely because `SCLP-815`'s more-aggressive health-check reinit increases the odds of two reconnection paths overlapping.

**Expected**: reconnection/reinit is idempotent — concurrent triggers should not crash the app.
**Actual**: two callers can race into joining the same cached `PhoenixChannel` object, and the second `.join()` call crashes with the assertion above.

## Root Cause

**File**: `syncro-flutter/lib/core/services/chat_websocket_service.dart`

**What it does (before fix)**:

1. `phoenix_socket`'s `PhoenixSocket.addChannel({required topic, ...})` (confirmed by reading `~/.pub-cache/hosted/pub.dev/phoenix_socket-0.7.7/lib/src/socket.dart:372-393`) caches channels by `topic` on the socket instance — calling it twice with the same topic returns the **same** `PhoenixChannel` object, not a new one.
2. `PhoenixChannel.join()` can only ever succeed once per channel instance (`assert(!_joinedOnce)`); a second `.join()` on the same object throws.
3. **`_tryInitialConnection()`** had no reentrancy guard. Four call sites could invoke it concurrently whenever `_isInitialized` was `false`: `init()`, `_handleNetworkReconnection()` (network-change listener), `getInteractions()`, `getMessagesOnInteractionWithId()`. A brief connectivity flap while the chat screen is actively polling is enough to trigger two overlapping calls, both eventually calling `.join()` on the same cached user-channel object.
4. **`_createAndJoinChannel()`** (message channel, used by `sendMessageToAssetId`, `markLastMessageAsRead`, `startNewChat`, `archiveInteraction`) had the identical unguarded race. `sendMessageToAssetId` is declared `void async` (fire-and-forget) and its only caller, `chat_detail_cubit.dart:150`, does not await it — so a UI double-tap on "send message" fires two concurrent calls for the same asset topic, hitting the same crash. The existing retry logic (recreate the whole socket on `!_joinedOnce`, from the SE-11997/SCLP-815 era) reacted to the crash after the fact by tearing down the shared socket — collateral damage to any other concurrent operation — instead of preventing the race.
5. Separately, 3 `onDone` stream-closed callbacks called `_handleNetworkReconnection()` directly instead of through `runZonedGuarded(_handleNetworkReconnection, _onZoneError)`, unlike every other call site (`init()`, the connectivity listener, the health-check timer) — no zone-level safety net if that call throws.

**What it should do**: only one initialization/join attempt should ever be in flight per shared resource; concurrent callers should wait their turn (or share the result, when safe) instead of racing.

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | modify | Added `_initializingCompleter` (single-flight guard on `_tryInitialConnection`), `_channelOperationLock` (turnstile guard on `_createAndJoinChannel`), and `runZonedGuarded` wrapping on the 3 `onDone` reconnection calls. |

### Do NOT Modify

- `syncro-flutter/lib/core/services/freebird_websocket_service.dart` (task-04's file — separate service, separate fix)
- `syncro-flutter/lib/features/authentication/login/presentation/login_web_view.dart` (task-05's reference-only file)

## Implementation Steps (already completed — documented retroactively)

### Step 1: Guard `_tryInitialConnection()`

Added `Completer<void>? _initializingCompleter`. All 4 callers target the same fixed user-channel topic (`account:user:{accountId}:{userId}`), so a **single-flight** pattern is safe: the first caller runs the real init; concurrent callers just await the same `Future`.

### Step 2: Guard `_createAndJoinChannel()`

Added `Future<void>? _channelOperationLock`. Unlike Step 1, each call can target a **different** asset topic — sharing a single cached result would incorrectly tell a waiter "your topic is ready" when a different topic was actually joined. Used a **turnstile** instead: each caller awaits the previous holder's completion, then re-checks `_needsNewChannel(topic)` (skip if another caller already joined the exact same topic while waiting), then runs its own attempt.

### Step 3: Zone-guard the 3 `onDone` reconnection calls

Wrapped `_handleNetworkReconnection()` in `runZonedGuarded(_handleNetworkReconnection, _onZoneError)` in the socket messageStream, user channel messages, and message channel messages `onDone` callbacks — matching the existing pattern in `init()`, the connectivity listener, and the health-check timer.

## Testing

- [x] `fvm dart analyze lib/core/services/chat_websocket_service.dart` — no issues found
- [x] `fvm flutter test test/core/services/chat_websocket_service_connect_test.dart test/features/chat/` — 56 tests passed, 0 regressions
- [ ] No new automated regression test for the race itself — same "untestable without refactoring" constraint already documented for this singleton class in task-04's `status.md` (singleton + field-init `GetIt` deps + no injection seam to simulate two concurrent callers deterministically). Verification is via:
  - Confirmed root cause by reading the actual `phoenix_socket` source (`~/.pub-cache/hosted/pub.dev/phoenix_socket-0.7.7/lib/src/socket.dart:372-393`), not guessing
  - Full existing test suite passing clean after the change
- [ ] Manual staging verification recommended before closing (double-tap "send message" rapidly on a live chat; toggle airplane mode on/off while the chat screen is open) — not yet performed

## Completion Criteria

- [x] Bug fixed
- [ ] Regression test added — not feasible without a refactor (see Testing above); accepted per this plan's existing precedent (task-04)
- [x] Existing tests pass
- [x] `pre-commit-check` passes (no hardcoded URLs/secrets, no `print()`/`debugPrint()`, `logger()` used throughout)
- [ ] Changes committed — pending (this task documents the fix; the actual commit happens in `syncro-flutter` immediately after this plan update, referencing `SE-13805 / fix-production-crashes-v180 task-09`)
- [x] Status updated in `status.md`
- [x] No blocked task to unblock

## Plan Updates Required

- `overview.md` Out of Scope (line 63): remove/correct the claim that this file is "already has guarded join/timeout/error-handling logic ... verified during the original planning" — that verification missed `_tryInitialConnection()` and `_createAndJoinChannel()`.
- `overview.md` Key Files table (line 116): remove "Reference only (already hardened) — out of scope, do not modify" for this file; it was modified by task-09.
- `overview.md` "Genuinely not yet planned by anyone" bullet (line 68) for `ChatWebSocketService.startNewChat` "Failed to join channel": resolved by task-09 — move out of that list.
- `overview.md` Task Summary, Crashlytics Issue IDs, Status line, Completion Checklist: add task-09 (see diffs applied to `overview.md`).
