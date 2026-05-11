# Task: Fix Unsafe Cast in ListChatsFromSocket.fromJson

**Plan**: SE-12379-chat-list-ios
**Task ID**: task-01
**Task Path**: task-01-fix-websocket-parsing
**Depends On**: None
**Ticket**: SE-12379

## Objective

Add a type guard in `ListChatsFromSocket.fromJson` so non-Map values in the WebSocket payload (e.g., `total_count` integers) are silently skipped instead of crashing the entire parse, preventing the chat list from ever being populated on iOS.

## Context

See [investigation.md](../investigation.md) Root Cause 1.

`_handleAddNewChat` in `chat_websocket_service.dart:541` calls `ListChatsFromSocket.fromJson(message.payload!)`. The `fromJson` at line 1111 iterates `json.values` and casts each value to `Map<String, dynamic>`. The `interactions_batched` payload includes a `total_count` integer key alongside chat objects; on iOS this integer is encountered first, causing a `TypeError` that silently drops the entire update.

The fix is one line: replace `.map((value) => ... value as Map...)` with `.whereType<Map<String, dynamic>>()` to skip non-Map entries before parsing.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read the ticket: https://syncrotech.atlassian.net/browse/SE-12379
- [ ] Read [investigation.md](../investigation.md) Root Cause 1
- [ ] Check `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/core/services/chat_websocket_service.dart` | modify | Fix `ListChatsFromSocket.fromJson` only — lines 1111–1119 |

### Do NOT Modify

- `lib/features/chat/application/chat_cubit.dart` — owned by task-04
- `lib/features/home/presentation/home_page.dart` — owned by task-03
- `lib/features/chat/application/new_chat_cubit.dart` — owned by task-02

## Implementation Steps

### Step 1: Open the file

`lib/core/services/chat_websocket_service.dart`

Locate the `ListChatsFromSocket.fromJson` factory at line 1111:

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

### Step 2: Apply the fix

Replace with:

```dart
factory ListChatsFromSocket.fromJson(Map<String, dynamic> json) {
  return ListChatsFromSocket(
    chats: json.values
        .whereType<Map<String, dynamic>>()
        .map(ChatFromSocket.fromJson)
        .toList(),
  );
}
```

`whereType<Map<String, dynamic>>()` filters out any non-Map entries (integers, strings, nulls) before parsing. This is safe because non-Map values are metadata, not chat objects.

### Step 3: Format and analyze

```bash
cd syncro-flutter
fvm dart format lib/core/services/chat_websocket_service.dart
fvm flutter analyze
```

Fix any analyzer warnings before proceeding.

## Testing

- [ ] `fvm flutter test` passes (all existing tests green)
- [ ] `pre-commit-check` passes
- [ ] No analyzer warnings

## Documentation / KB Updates

- [ ] No KB/doc updates required — the fix is a one-line type guard with no API or behavioral changes visible to consumers.

## Completion Criteria

- [ ] `ListChatsFromSocket.fromJson` uses `.whereType<Map<String, dynamic>>()` before `.map`
- [ ] `fvm flutter analyze` clean
- [ ] All existing tests pass
- [ ] Changes committed to `plan/SE-12379-chat-list-ios/task-01-fix-websocket-parsing` branch
- [ ] Status updated in `status.md`
