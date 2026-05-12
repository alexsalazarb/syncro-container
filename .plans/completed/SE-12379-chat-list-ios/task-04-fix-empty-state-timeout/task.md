# Task: Fix Empty State Flash and Timeout Coverage in refresh()

**Plan**: SE-12379-chat-list-ios
**Task ID**: task-04
**Task Path**: task-04-fix-empty-state-timeout
**Depends On**: None
**Ticket**: SE-12379

## Objective

Fix `ChatCubit.refresh(showLoading: true)` so it does not immediately show an empty "No chats" state before the WebSocket responds, and ensure the safety-net timeout covers the `ChatLoaded([])` state so the user sees an error instead of a permanently empty list when WebSocket delivery fails.

## Context

See [investigation.md](../investigation.md) Root Cause 2.

`lib/features/chat/application/chat_cubit.dart:342`:

```dart
Future<void> refresh({bool showLoading = true}) async {
  if (showLoading) {
    safeEmit(const ChatLoaded(chats: []));  // ← shows "No chats yet" immediately
    await getChats(showLoading: showLoading);
  }
  ...
}
```

`_startResponseTimer` at line 326:

```dart
_responseTimer = Timer(_responseTimeout, () {
  if (!isClosed && (state is ChatLoading || state is ChatInitial)) {
    safeEmit(const ChatError(...));  // ← doesn't cover ChatLoaded([])
  }
});
```

Two fixes needed:
1. Change `safeEmit(const ChatLoaded(chats: []))` to `safeEmit(ChatLoading())` so a loading spinner is shown instead of an empty state.
2. Update the timer check to also cover `ChatLoaded` with an empty list.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read the ticket: https://syncrotech.atlassian.net/browse/SE-12379
- [ ] Read [investigation.md](../investigation.md) Root Cause 2
- [ ] Check `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read `lib/features/chat/application/chat_cubit.dart` lines 280–360 in full before writing
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/chat/application/chat_cubit.dart` | modify | Fix `refresh()` and `_startResponseTimer()` only |

### Do NOT Modify

- `lib/core/services/chat_websocket_service.dart` — owned by task-01
- `lib/features/chat/application/new_chat_cubit.dart` — owned by task-02
- `lib/features/home/presentation/home_page.dart` — owned by task-03

## Implementation Steps

### Step 1: Fix the empty state in refresh()

Locate `refresh()` around line 342. Change:

```dart
if (showLoading) {
  safeEmit(const ChatLoaded(chats: []));
  await getChats(showLoading: showLoading);
}
```

To:

```dart
if (showLoading) {
  safeEmit(ChatLoading());
  await getChats(showLoading: showLoading);
}
```

`ChatLoading()` shows a spinner instead of "No chats yet". `getChats(showLoading: true)` also emits `ChatLoading()` at its entry, so this is consistent. Verify `ChatLoading` is a valid state — check `chat_state.dart`.

### Step 2: Fix the safety-net timer

Locate `_startResponseTimer()` around line 326. Change:

```dart
if (!isClosed && (state is ChatLoading || state is ChatInitial)) {
  safeEmit(const ChatError(...));
}
```

To:

```dart
if (!isClosed &&
    (state is ChatLoading ||
        state is ChatInitial ||
        (state is ChatLoaded && (state as ChatLoaded).chats.isEmpty))) {
  safeEmit(const ChatError(...));
}
```

This ensures that if the WebSocket never delivers data and the list is still empty when the timer fires, the user sees an error state (with retry) rather than a permanently empty list.

### Step 3: Format and analyze

```bash
cd syncro-flutter
fvm dart format lib/features/chat/application/chat_cubit.dart
fvm flutter analyze
```

## Testing

- [ ] `fvm flutter test` passes (all existing tests green)
- [ ] Existing `ChatCubit` tests still pass — check for any tests asserting `ChatLoaded([])` is emitted by `refresh()` and update them to expect `ChatLoading()` instead
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required.

## Completion Criteria

- [ ] `refresh(showLoading: true)` emits `ChatLoading()` instead of `ChatLoaded(chats: [])`
- [ ] `_startResponseTimer` covers `ChatLoaded(chats: [])` in its condition
- [ ] All existing `ChatCubit` tests pass (update any that assert the old empty-state behavior)
- [ ] `fvm flutter analyze` clean
- [ ] Changes committed to `plan/SE-12379-chat-list-ios/task-04-fix-empty-state-timeout` branch
- [ ] Status updated in `status.md`
