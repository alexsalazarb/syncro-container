# Interaction / Chat Domain Concept

**Last Updated**: April 2026

## Context

An Interaction (also called a Chat) is a real-time text conversation between a technician and an end-user about a specific managed asset.

## Core Properties

- `id` — numeric interaction ID
- `assetId` — the asset this conversation is about
- `userId` — technician handling the conversation
- `accountId` — MSP account
- `userLastReadAt` — when the technician last read the conversation
- `assetLastReadAt` — when the end-user last read
- `lastMessageInsertedAt` — timestamp of most recent message
- `archivedAt` — set when the conversation is closed/archived

## Lifecycle

```
Open (active) → Archived (closed)
```

Active interactions are fetched via `get_interactions_batched` Phoenix event with scope `"open"`. Archived interactions use scope `"archived"`.

## Unread Detection

An interaction has unread messages if:
```
lastMessageInsertedAt > userLastReadAt
```

## Relationship to Asset

Every interaction is 1:1 with an asset. A customer can only have one active (open) interaction per asset at a time.

## Flutter Feature

- `chat` feature — list of interactions
- `chat_detail` feature — individual conversation view
- Transport layer: `ChatWebSocketService` (Phoenix WebSocket)

See `docs/kb-projects/syncro-flutter/product/features/real-time-chat.md` and `docs/kb-projects/syncro-flutter/technical/integrations/phoenix-socket.md`.
