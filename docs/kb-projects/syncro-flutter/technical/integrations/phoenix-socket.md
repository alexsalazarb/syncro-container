# Phoenix Socket Chat Integration — syncro-flutter

**Last Updated**: June 2026

## Context

Real-time chat between technicians and end-users uses an Elixir/Phoenix backend accessed via WebSocket. The `phoenix_socket` Dart package manages the connection.

## Architecture

`ChatWebSocketService` is a **singleton** (factory constructor pattern). It manages two types of Phoenix channels:

| Channel | Topic Pattern | Purpose |
|---------|--------------|---------|
| User channel | `account:user:{accountId}:{userId}` | Receives all interaction/chat events for the technician |
| Message channel | `account:asset:{accountId}:{assetId}` | Per-asset channel for sending/receiving messages in a specific chat |

## Streams (Exposed for Cubits)

```dart
Stream<ChatFromSocket?> chatSocketStream            // single chat update
Stream<List<ChatFromSocket>?> chatsSocketStream     // full chat list update
Stream<MessageFromSocket?> messagesSocketStream     // new message received
Stream<ChatFromSocket?> newChatSocketStream         // new interaction created
```

Cubits listen to these streams directly. Always cancel subscriptions in `close()`.

## Connection Lifecycle

1. `init()` — Only initializes if authenticated. Sets up connectivity monitoring.
2. `_tryInitialConnection()` — Gets chat JWT from REST API, creates PhoenixSocket, connects, joins user channel.
3. `_startHealthCheck()` — Periodic 5-minute check of socket and channel state.
4. Network lost → `_handleNetworkReconnection()` → full reinit.
5. `disconnect()` — Called on logout.

## Chat Token

The chat WebSocket uses a separate JWT obtained from the REST API:

```
GET /api/v1/chat/access_token  →  { "token": "..." }
```

URL: `wss://chat-chat.{basePath}/socket/websocket?token={chatToken}`

## Key Operations

```dart
// Get list of open chat interactions
await chatWebSocketService.getInteractions("open");

// Send a message
chatWebSocketService.sendMessageToAssetId(interactionId, assetId, message);

// Mark as read
await chatWebSocketService.markLastMessageAsRead(interactionId, assetId);

// Start a new chat
await chatWebSocketService.startNewChat(assetId);

// Archive a chat
final success = await chatWebSocketService.archiveInteraction(
  interactionId, asset, archiveName,
);
```

## Error Handling Pattern

The service uses a `_logErrorOnce()` throttle (30-second cooldown) to avoid log spam during connection outages. Unexpected errors (not network-related) are reported to Crashlytics.

Phoenix channel state errors (`!_joinedOnce`) trigger a socket recreation flow.

## wasChatArchived Flag

When a chat is archived, `wasChatArchived = true` is set **before** calling archive. The socket message listener checks this flag to detect if the next channel-join reply is for a new interaction vs. a history resume. It resets to `false` after processing.

## Reconnection Strategy

- Up to 5 reconnect attempts for the socket
- Exponential backoff: 2s, 4s, ... between attempts
- Connectivity change triggers full reinit (not just reconnect)
- `Completer`-based serialization ensures only one `connect()` runs at a time

---

## Gotchas

### `_createListeners()` can be called more than once — always cancel before re-subscribing

`_tryInitialConnection()` calls `_createListeners()` to attach listeners to `_socket.messageStream` and `_channel.messages`. It can be invoked more than once:

1. From `ChatWebSocketService.init()` on login (called with `unawaited()` — fire-and-forget)
2. From `getInteractions()` fallback when `_channel?.canPush == false` at call time

If the second path runs before the first completes, **two sets of listeners accumulate**. Every channel event then fires all listeners, causing `_handleChatUpdate` to execute multiple times per `interactions_batched` event.

**The fix** (SE-12760): store both subscriptions in `StreamSubscription?` fields and cancel them at the top of `_createListeners()` before re-subscribing:

```dart
StreamSubscription? _socketMessageSubscription;
StreamSubscription? _channelMessageSubscription;

void _createListeners() {
  _socketMessageSubscription?.cancel();
  _channelMessageSubscription?.cancel();

  _socketMessageSubscription = _socket?.messageStream.listen(/* ... */);
  _channelMessageSubscription = _channel!.messages.listen(/* ... */);
}
```

Always cancel both subscriptions in `dispose()` as well. **Never add new `.listen()` calls inside `_createListeners()` without first cancelling the previous ones.**

### `unawaited(init())` creates a race with early consumers

`route_cubit.dart` calls `unawaited(chatWebSocketService.init())` at login and immediately navigates to Home. If any cubit (`ChatCubit`, etc.) calls `getInteractions()` before `init()` completes, the `_channel?.canPush == false` fallback triggers a second `_tryInitialConnection()`. This is the specific race that caused the duplicate listener bug in SE-12760 (visible chat list flicker).

### `!_joinedOnce` assertion from phoenix_socket on hot restart

Occurs when `addChannel()` + `join()` is called on a channel already in the `joined` state (e.g., after a Dart hot restart without full reconnection). The fix is a **full process restart** (`flutter run`), not just hot restart. This error is caught, logged to Crashlytics, and triggers `disconnect()` internally.

### `dart:developer log()` is silent in `flutter run` stdout

Use `debugPrint()` (from `package:flutter/foundation.dart`) for any diagnostic output to the terminal. `dev.log()` goes through a different pipeline and produces no visible output in `flutter run` stdout.

### `interactions_batched` payload structure

The server response to `get_interactions_batched` is a JSON **object** (not array), keyed by index strings plus a `total_count` int:

```json
{
  "response": {
    "total_count": 3,
    "0": { "id": 1, "asset_id": 10, "last_message": { "inserted_at": "..." }, ... },
    "1": { ... },
    "2": { ... }
  }
}
```

`ListChatsFromSocket.fromJson` uses `.whereType<Map<String, dynamic>>()` to filter out the `total_count` integer. `last_message.inserted_at` is the **only sortable timestamp** in the payload — there is no `updated_at` or `created_at` at the interaction level.
