# Task: Fix Duplicate WebSocket Listeners in ChatWebSocketService

**Plan**: SE-12760-chat-sort-ios
**Task ID**: task-05
**Task Path**: task-05-fix-duplicate-listeners
**Depends On**: None (independent — fixes the race condition root cause)
**Ticket**: SE-12760

## Objective

Fix `ChatWebSocketService._createListeners()` to cancel previous subscriptions before attaching new ones. This eliminates the duplicate listener accumulation that causes `_handleChatUpdate` to fire twice per `interactions_batched` event, which is the root cause of the visible flicker (list shows in one order, immediately reorders).

## Root Cause

`_createListeners()` subscribes to two streams without storing the returned `StreamSubscription` objects:

```dart
void _createListeners() {
  _socket?.messageStream.listen(/* handler */);   // subscription discarded
  _channel!.messages.listen(/* handler */);       // subscription discarded
}
```

`_tryInitialConnection()` calls `_createListeners()` and can be invoked more than once:
1. From `ChatWebSocketService.init()` — called from `route_cubit.dart:97` with `unawaited()` (fire-and-forget)
2. From `getInteractions()` fallback reinit — triggered when `_channel?.canPush` is false

**Race**: navigation to Chat tab happens before `init()` completes → `getChats()` → `getInteractions()` finds `_isInitialized = false` → calls `_tryInitialConnection()` again → second `_createListeners()` → two listeners accumulate → double event delivery.

## Fix

### Step 1: Add subscription fields to `ChatWebSocketService`

In the subscription fields section (around line 128), add:

```dart
StreamSubscription? _socketMessageSubscription;
StreamSubscription? _channelMessageSubscription;
```

### Step 2: Update `_createListeners()` to cancel before re-subscribing

Replace the bare `.listen()` calls with stored subscriptions:

```dart
void _createListeners() {
  _socketMessageSubscription?.cancel();
  _channelMessageSubscription?.cancel();

  _socketMessageSubscription = _socket?.messageStream.listen(
    /* existing handler — no changes */
  );

  _channelMessageSubscription = _channel!.messages.listen(
    /* existing handler — no changes */
  );
}
```

### Step 3: Cancel in `dispose()`

In `dispose()`, add cancellation before closing controllers:

```dart
void dispose() {
  disconnect();
  _healthCheckTimer?.cancel();
  _connectivitySubscription?.cancel();
  _socketMessageSubscription?.cancel();
  _channelMessageSubscription?.cancel();
  _connectivityService.dispose();
  // ... close controllers
}
```

## File Ownership

- `lib/core/services/chat_websocket_service.dart` — only this file changes

**Do NOT modify**: `chat_cubit.dart`, `route_cubit.dart`, any test files (task-04 handles tests)

## Testing

- [ ] `fvm flutter analyze` — no errors
- [ ] Navigate to Chats tab: confirm NO visible flicker (list appears once, stable)
- [ ] Open a chat and navigate back: confirm chat list re-loads without flicker
- [ ] Hot restart: confirm `!_joinedOnce` assertion error does NOT occur (socket reinit is clean)
- [ ] Run existing `ChatWebSocketService` tests: `fvm flutter test --name "ChatWebSocketService"`

## Completion Criteria

- [ ] `_socketMessageSubscription` and `_channelMessageSubscription` fields added
- [ ] `_createListeners()` cancels previous subscriptions before attaching new ones
- [ ] `dispose()` cancels both subscriptions
- [ ] No flicker visible when navigating to Chats tab on simulator
- [ ] `fvm flutter analyze` passes with no warnings
- [ ] Status updated in `status.md`
