# Task: Fix Chat List Not Refreshing on AppBar Back from Chat Detail

**Plan**: SE-12760-chat-sort-ios
**Task ID**: task-06
**Task Path**: task-06-fix-back-navigation-refresh
**Depends On**: None
**Ticket**: SE-12760

## Objective

Fix `chat_detail_page.dart` so that `leaveMessageChannel()` is always called when navigating back via the AppBar back button — not just when `isNewChat` is true. This ensures the chat list refreshes after sending a message, so the updated chat appears at the top on return.

## Root Cause

`ChatDetailPage` has two back navigation paths:

| Path | Trigger | `leaveMessageChannel()` called? |
|------|---------|--------------------------------|
| `PopScope.onPopInvokedWithResult` | System back / iOS swipe gesture | Always ✓ |
| `onBackPressed` | AppBar `‹` button | Only when `isNewChat == true` ✗ |

`goBack()` → `state.pop()` is programmatic and **bypasses `PopScope`**. For existing chats using the AppBar button, `leaveMessageChannel()` is never called, so `ChatCubit.getChats()` is never triggered, and the server never pushes an updated `interactions_batched` list.

`leaveMessageChannel()` is appropriate for ALL back navigations, not just new chats:
```dart
Future<void> leaveMessageChannel() async {
  _chatWebSocketService.leaveMessageChannel();  // leaves per-asset message channel
  await getChats();                              // triggers fresh getInteractions push
}
```

## Fix

In `lib/features/chat_detail/presentation/chat_detail_page.dart`, remove the `isNewChat` condition from `onBackPressed`:

**Before:**
```dart
onBackPressed: () {
  if (parameters.isNewChat) {
    context.read<ChatCubit>().leaveMessageChannel();
  }
  context.read<RouteCubit>().goBack();
},
```

**After:**
```dart
onBackPressed: () {
  context.read<ChatCubit>().leaveMessageChannel();
  context.read<RouteCubit>().goBack();
},
```

## File Ownership

- `lib/features/chat_detail/presentation/chat_detail_page.dart` — only this file changes

**Do NOT modify**: `chat_cubit.dart`, `chat_websocket_service.dart`, `route_cubit.dart`

## Testing

- [ ] `fvm flutter analyze lib/features/chat_detail/presentation/chat_detail_page.dart`
- [ ] Open an existing chat → send a message → press AppBar `‹` → confirm the chat appears at the top of the list
- [ ] Open an existing chat → press AppBar `‹` without sending a message → list still loads correctly (no regression)
- [ ] Open a new chat → press AppBar `‹` → same behavior as before (no regression)
- [ ] System back / iOS swipe → same behavior as before (no regression)
- [ ] Run chat-related tests: `fvm flutter test test/features/chat/ test/features/chats/`

## Completion Criteria

- [ ] `onBackPressed` calls `leaveMessageChannel()` unconditionally
- [ ] `fvm flutter analyze` passes with no warnings
- [ ] All existing chat tests pass
- [ ] Status updated in `status.md`
