# Status: Diagnostic Logging — iOS Chat Sort Investigation

**Current Status**: adapted
**Last Updated**: 2026-06-17
**Agent**: Claude Code + Alex Salazar
**Branch**: feature/SE-12530-flutter-upgrade
**PR**: —

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-06-16 | not-started | Task created |
| 2026-06-16 | in-progress | Adapted: Charles Proxy not available → used `debugPrint` on own device via simulator |
| 2026-06-17 | adapted | Investigation complete. Logging added, payload captured, root cause identified. See findings below. |

## What Was Found

### Payload structure (interactions_batched)

Each chat entry in the socket payload has:
- `id`, `account_id`, `asset_id`, `user_id`
- `remove_history`, `archived_at`
- `user_last_read_at`, `asset_last_read_at`
- `last_message.inserted_at` — **the only sortable timestamp in the payload**
- No `updated_at`, no `created_at`, no alternative timestamp fields

### Double-firing discovered

`_handleChatUpdate` fires **twice** on every navigation to Chats. Logs:
```
[SE-12760] Raw socket payload (7 chats):  ← call #1
[SE-12760] Raw socket payload (7 chats):  ← call #2
[SE-12760] After sort (7 chats):          ← call #1 sort
[SE-12760] After sort (7 chats):          ← call #2 sort
```

### Root cause: duplicate listeners in `_createListeners()`

`ChatWebSocketService._createListeners()` subscribes to `_socket.messageStream` and `_channel!.messages` WITHOUT storing or canceling previous subscriptions. When called more than once (triggered by the race between `unawaited(chatWebSocketService.init())` in `route_cubit.dart:97` and `ChatCubit.getChats()` → `getInteractions()` fallback reinit), duplicate listeners accumulate. Each `interactions_batched` event fires all listeners → multiple `_handleChatUpdate` calls.

### Customer video observation

The visible flicker (list shows in one order, immediately reorders) matches the double `_handleChatUpdate` firing. Two concurrent `getChatInformationByIdsUseCase` async calls return in different orders → different `chatsWithAssetInfo` subsets → different `ChatLoaded` emissions in rapid succession.

Final order wrong: server `inserted_at` values may not match web app sort order for account SA 75758. Requires backend confirmation after mobile fix.

## Blockers

None — investigation complete. Logging removed.

## Artifacts

- Diagnostic logs captured during session via `debugPrint` on iPhone 17 Pro simulator
- Payload structure documented above
- Logging removed from both `chat_cubit.dart` and `chat_websocket_service.dart`

## Adaptations

- **Charles Proxy → debugPrint**: No proxy tool available; used Dart `debugPrint` on own device in simulator instead.
- **New task added**: task-05 added for the duplicate listener fix (root cause of flicker / double-firing).
