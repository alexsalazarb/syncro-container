# Phoenix Socket Chat Integration — syncro-flutter

**Last Updated**: April 2026

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
