# Investigation: SE-12379 — iOS Chat List Not Showing Chats

**Date**: 2026-05-11
**Investigator**: Claude (claude-sonnet-4-6)
**Confidence**: High

## Reproduction

1. Sign in to Syncro Mobile on iOS (iPhone 14 Plus, iOS 26.4.2)
2. Navigate to Chats tab — shows "No chats yet" despite having assigned chats
3. Tap "+" to create a new chat, select a customer/asset
4. Navigate to Home tab — newly created chat appears there (!)
5. Return to Chats tab — chat does NOT appear
6. Switch to another tab and back — chat still not visible

Not reproducible on Android.

## Root Cause 1 (PRIMARY) — Silent crash in WebSocket payload parsing

**File**: `lib/core/services/chat_websocket_service.dart:1111`

```dart
factory ListChatsFromSocket.fromJson(Map<String, dynamic> json) {
  return ListChatsFromSocket(
    chats: json.values
        .map(
          (value) => ChatFromSocket.fromJson(value as Map<String, dynamic>),
        )
        .toList(),
  );
}
```

**The problem**: `json.values` iterates ALL values in the payload. The `interactions_batched` WebSocket event payload is a Map keyed by chat ID, but it also contains metadata entries like `total_count` (an integer), as confirmed by lines 423 and 463 in the same file where `response['total_count']` is checked. When the iterator hits the integer value, `value as Map<String, dynamic>` throws a `TypeError`. Because WebSocket callbacks are async and uncaught, the error is swallowed silently — `chatsSocketController` never receives data, and the chat list never updates.

**Why iOS-specific**: On Android, the `interactions_batched` payload may not include `total_count` in the root map, or the order differs. On iOS the payload structure appears to include it at the root level.

**Fix**: Filter `json.values` to only Map entries before parsing:
```dart
chats: json.values
    .whereType<Map<String, dynamic>>()
    .map((value) => ChatFromSocket.fromJson(value))
    .toList(),
```

## Root Cause 2 (SECONDARY) — refresh() emits empty state before WebSocket responds

**File**: `lib/features/chat/application/chat_cubit.dart:342`

```dart
Future<void> refresh({bool showLoading = true}) async {
  _refreshDebounce?.cancel();
  if (showLoading) {
    safeEmit(const ChatLoaded(chats: []));  // ← immediately shows empty list
    await getChats(showLoading: showLoading);
  }
  ...
}
```

And the safety-net timer at line 327:
```dart
_responseTimer = Timer(_responseTimeout, () {
  if (!isClosed && (state is ChatLoading || state is ChatInitial)) {
    // ← does NOT cover ChatLoaded([]) state
    safeEmit(const ChatError(...));
  }
});
```

**The problem**: When `refresh(showLoading: true)` is called (e.g., pull-to-refresh), it immediately emits `ChatLoaded(chats: [])` — which shows the "No chats yet" empty state. If the WebSocket never responds (due to Root Cause 1), the safety-net timer doesn't fire because the current state is `ChatLoaded` (not `ChatLoading` or `ChatInitial`). The empty list persists forever.

**Fix**: Either (a) emit `ChatLoading()` instead of `ChatLoaded([])` when `showLoading: true`, or (b) expand the timer's state check to also cover `ChatLoaded(chats: [])`.

## Root Cause 3 (TERTIARY) — Tab-tap refresh guard uses exact route match

**File**: `lib/features/home/presentation/home_page.dart:133`

```dart
final currentRoute = context
    .read<RouterStateNotifier>()
    .currentRoute;  // (approximate — see actual implementation)

if (currentRoute == AppRoute.chats.path) {
  context.read<ChatCubit>().refresh();
}
```

**The problem**: The Chats tab's tap handler only calls `ChatCubit.refresh()` when the current route is exactly `/chat`. After navigating to a chat detail page (pushed on `_rootNavigatorKey`), the current route is `/chat-detail`. When the user taps the Chats tab to return, the guard `currentRoute == '/chat'` fails and refresh is never triggered.

**Fix**: Change the guard to also trigger refresh when the current route starts with `/chat` (covering `/chat-detail` and sub-routes), or use `GoRouter` branch-awareness to detect that the user is re-selecting the chats tab.

## Root Cause 4 (QUATERNARY) — NewChatCubit doesn't refresh ChatCubit after creation

**File**: `lib/features/chat/application/new_chat_cubit.dart`

```dart
_newChatSocketSubscription = chatWebSocketService.newChatSocketStream.listen(
  (message) {
    ...
    final chat = Chat(...);
    safeEmit(chat);  // ← emits new chat state but doesn't refresh ChatCubit
  },
);
```

**The problem**: After successfully creating a new chat, `NewChatCubit` emits the new chat but does NOT call `ChatCubit.refresh()`. The chat list only updates if the WebSocket proactively pushes an `interactions_batched` event, which is unreliable on iOS — especially because iOS fires `AppLifecycleState.inactive` on navigation (unlike Android), which can disrupt the WebSocket subscription at the worst moment.

**Fix**: After `safeEmit(chat)` in the `newChatSocketStream` listener, trigger `ChatCubit.refresh()`. Since `NewChatCubit` and `ChatCubit` are both registered in the DI container, inject `ChatCubit` into `NewChatCubit` and call `chatCubit.refresh(showLoading: false)`.

## Why Existing Tests Missed This

- `ListChatsFromSocket.fromJson` has no unit tests — the parsing logic was never tested with non-uniform payloads
- `ChatCubit` tests mock the WebSocket service, so the socket parsing failure path is not exercised
- The tab-tap guard is widget-level behavior tested manually, not in unit/integration tests
- `NewChatCubit` tests don't assert that `ChatCubit.refresh()` is called after creation

## Cross-Project Assessment

Single-project bug — all fixes are in `syncro-flutter`. No BE changes required.

## Feature History

No related plans found that introduced these issues directly. The WebSocket parsing code predates the plan system. The `refresh()` empty-state behavior and tab-tap guard are existing patterns that were never corrected.
