# Real-time Chat Feature — syncro-flutter

**Last Updated**: April 2026

## Context

Technicians can chat with end-users in real-time. Chat is asset-based — each conversation is associated with a specific managed device (asset).

## Architecture

- **Transport**: Phoenix WebSocket (see `technical/integrations/phoenix-socket.md`)
- **Chat list**: `ChatCubit` — subscribes to `chatsSocketStream`, loads chat metadata
- **Chat detail**: `ChatDetailPage` with `ChatDetailCubit` — subscribes to `messagesSocketStream`
- **Filter**: `ChatFilterCubit` — filters chats by status (open/closed)

## Navigation

Chat detail is accessible from two places:
1. Chat tab → `ChatPage` → `ChatDetailPage` (via `AppRoute.chatDetail`)
2. Push notification (source `openStruct`) → direct to `ChatDetailPage`

`ChatDetailPage` receives a `ChatDetailParameters` object via `GoRouter.extra`:

```dart
class ChatDetailParameters {
  final Chat chat;
  final Asset asset;
}
```

## Chat Lifecycle

1. Open chat: `startNewChat(assetId)` → joins `account:asset:{accountId}:{assetId}` channel
2. Load messages: `getMessagesOnInteractionWithId(interactionId)` → push event on socket
3. Send message: `sendMessageToAssetId(interactionId, assetId, message)`
4. Mark as read: `markLastMessageAsRead(interactionId, assetId)`
5. Close/archive: `archiveInteraction(interactionId, asset, archiveName)`
6. Leave channel: `leaveMessageChannel()` — called on page dispose

## Create Ticket from Chat

From `ChatDetailPage`, technicians can create a ticket linked to the chat's asset/customer. This navigates to `TicketCreatePage` via `AppRoute.ticketCreateFromChat` (a root-level route outside the shell).

## Unread Count

Unread status is determined by comparing `userLastReadAt` vs `lastMessageInsertedAt` on `ChatFromSocket`. The `ChatCubit` computes unread counts from socket data.

## Known Behaviors

- `ChatCubit` listens to both `TechniciansCubit` (for assignee names) and `ChatFilterCubit` (for status filtering)
- `NewChatCubit` handles the "initiate new chat" flow from the asset detail screen
- Chat list uses `chatsSocketStream` which delivers full list updates (not incremental diffs)
