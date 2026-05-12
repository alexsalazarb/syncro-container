# Task: Refresh Chat List from NewChatCubit After Creation

**Plan**: SE-12379-chat-list-ios
**Task ID**: task-02
**Task Path**: task-02-fix-new-chat-refresh
**Depends On**: None
**Ticket**: SE-12379

## Objective

After a new chat is successfully created, trigger `ChatCubit.refresh()` so the chat list updates immediately without relying on the WebSocket proactively pushing an `interactions_batched` event — which is unreliable on iOS.

## Context

See [investigation.md](../investigation.md) Root Cause 4.

`NewChatCubit` (`lib/features/chat/application/new_chat_cubit.dart`) listens to `chatWebSocketService.newChatSocketStream`. When a new chat is confirmed, it calls `safeEmit(chat)` but does not trigger a chat list refresh. On iOS, the WebSocket `interactions_batched` push that would update the list is unreliable because iOS fires `AppLifecycleState.inactive` on navigation, which can interrupt the WebSocket subscription.

The fix is to inject `ChatCubit` into `NewChatCubit` and call `chatCubit.refresh(showLoading: false)` after `safeEmit(chat)`.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read the ticket: https://syncrotech.atlassian.net/browse/SE-12379
- [ ] Read [investigation.md](../investigation.md) Root Cause 4
- [ ] Check `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read `lib/features/chat/application/new_chat_cubit.dart` in full before writing
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/chat/application/new_chat_cubit.dart` | modify | Inject ChatCubit, call refresh after emit |

### Do NOT Modify

- `lib/core/services/chat_websocket_service.dart` — owned by task-01
- `lib/features/home/presentation/home_page.dart` — owned by task-03
- `lib/features/chat/application/chat_cubit.dart` — owned by task-04

## Implementation Steps

### Step 1: Read the current NewChatCubit

Open `lib/features/chat/application/new_chat_cubit.dart` and understand:
- How `NewChatCubit` is constructed (GetIt injection? constructor params?)
- Where `safeEmit(chat)` is called in the `newChatSocketStream` listener

### Step 2: Inject ChatCubit

Add a `ChatCubit` parameter to `NewChatCubit`'s constructor (or retrieve it via `GetIt` if that's how the cubit is already obtaining services). Follow the existing pattern in the file — if `ChatWebSocketService` is obtained via `GetIt.instance`, do the same for `ChatCubit`:

```dart
final ChatCubit _chatCubit = GetIt.instance<ChatCubit>();
```

Or pass it as a constructor parameter if `NewChatCubit` is provided via BlocProvider. Check how it's provided in the widget tree before deciding — match the existing pattern.

### Step 3: Call refresh after emit

In the `newChatSocketStream` listener, after `safeEmit(chat)`, add:

```dart
safeEmit(chat);
_chatCubit.refresh(showLoading: false);
```

`showLoading: false` is intentional — the chat detail is already open, so a full loading spinner on the list behind it would be jarring. A silent background refresh is sufficient.

### Step 4: Format and analyze

```bash
cd syncro-flutter
fvm dart format lib/features/chat/application/new_chat_cubit.dart
fvm flutter analyze
```

Fix any analyzer warnings before proceeding.

## Testing

- [ ] `fvm flutter test` passes (all existing tests green)
- [ ] `pre-commit-check` passes
- [ ] No analyzer warnings

## Documentation / KB Updates

- [ ] No KB/doc updates required — this is a behavior fix with no new public API.

## Completion Criteria

- [ ] `NewChatCubit` calls `chatCubit.refresh(showLoading: false)` after `safeEmit(chat)` in the `newChatSocketStream` listener
- [ ] `fvm flutter analyze` clean
- [ ] All existing tests pass
- [ ] Changes committed to `plan/SE-12379-chat-list-ios/task-02-fix-new-chat-refresh` branch
- [ ] Status updated in `status.md`
