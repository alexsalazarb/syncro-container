# Task: Resilient fromJson for Null last_message

**Plan**: SE-12760-chat-sort-ios
**Task ID**: task-02
**Task Path**: task-02-fix-null-last-message
**Depends On**: None
**Ticket**: SE-12760

## Objective

Make `ChatFromSocket.fromJson` resilient to `last_message: null` (or `last_message` being absent) in the socket payload. Currently, a null `last_message` throws a `TypeError` that silently crashes the entire list parse — all chats disappear. Replace with a safe fallback that skips the offending entry and logs a warning.

## Context

`ListChatsFromSocket.fromJson` uses `.map(ChatFromSocket.fromJson).toList()`. If any one entry throws, the entire `.toList()` call throws — no chats are emitted. Since there is no per-item error handling, a single chat with `last_message: null` wipes the whole list.

This is a **latent P1 crash risk** for accounts with archived, deleted, or draft chats that have no messages. It is NOT the direct cause of the wrong-order bug in SE-12760 (chats ARE showing), but it MUST be fixed defensively.

See [investigation.md](../investigation.md) Finding 3.

**Files changed**: `chat_websocket_service.dart` only. No cubit or presentation layer changes.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [investigation.md](../investigation.md) Finding 3
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/core/services/chat_websocket_service.dart` | modify | `ChatFromSocket.fromJson` — add null guard on `last_message` |

### Do NOT Modify

- `lib/features/chat/application/chat_cubit.dart` — owned by task-01 and task-03

## Implementation Steps

### Step 1: Add null guard to `ChatFromSocket.fromJson`

Current (line ~1169):
```dart
lastMessageInsertedAt: DateTime.parse(
  '${(json['last_message'] as Map<String, dynamic>)['inserted_at']}Z',
),
```

Replace with:
```dart
lastMessageInsertedAt: _parseLastMessageInsertedAt(json['last_message']),
```

Add static helper below `fromJson`:
```dart
static DateTime _parseLastMessageInsertedAt(dynamic lastMessage) {
  if (lastMessage == null) return DateTime.fromMillisecondsSinceEpoch(0);
  final map = lastMessage as Map<String, dynamic>;
  final insertedAt = map['inserted_at'] as String?;
  if (insertedAt == null) return DateTime.fromMillisecondsSinceEpoch(0);
  return DateTime.parse('${insertedAt}Z');
}
```

### Step 2: Handle parsing failure gracefully in `ListChatsFromSocket.fromJson`

Current:
```dart
chats: json.values
    .whereType<Map<String, dynamic>>()
    .map(ChatFromSocket.fromJson)
    .toList(),
```

Wrap each `fromJson` call to skip malformed entries instead of crashing the whole list:
```dart
chats: json.values
    .whereType<Map<String, dynamic>>()
    .expand((entry) {
      try {
        return [ChatFromSocket.fromJson(entry)];
      } catch (e) {
        debugPrint('[ChatWebSocket] Skipping malformed chat entry: $e');
        return <ChatFromSocket>[];
      }
    })
    .toList(),
```

This ensures one bad entry never kills the whole list.

## Testing

- [ ] `ChatFromSocket.fromJson({'last_message': null, ...})` does not throw — returns epoch timestamp
- [ ] `ChatFromSocket.fromJson({'last_message': {}, ...})` does not throw — returns epoch timestamp
- [ ] `ListChatsFromSocket.fromJson` with one malformed entry returns the valid remaining chats
- [ ] Existing parsing tests still pass: `fvm flutter test test/core/services/`
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required
- [ ] No public API contract changes — internal parsing only

## Completion Criteria

- [ ] `ChatFromSocket.fromJson` does not throw on null or empty `last_message`
- [ ] `ListChatsFromSocket.fromJson` skips malformed entries instead of crashing
- [ ] Regression test added in task-04 covers both failure modes
- [ ] `fvm flutter analyze` passes with no new warnings
- [ ] Changes committed to `plan/SE-12760-chat-sort-ios/task-02-fix-null-last-message` branch
- [ ] Status updated in `status.md`
