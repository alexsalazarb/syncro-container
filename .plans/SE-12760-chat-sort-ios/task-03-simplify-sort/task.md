# Task: Simplify _sortChats Double-Reversal

**Plan**: SE-12760-chat-sort-ios
**Task ID**: task-03
**Task Path**: task-03-simplify-sort
**Depends On**: None
**Ticket**: SE-12760

## Objective

Replace the confusing double-reversal pattern in `_sortChats()` with a single-pass descending sort. No behavior change — this is a readability fix that prevents future misreads of the sort direction.

## Context

`_sortChats()` uses parameters named `(b, a)` (reversed) then calls `.reversed.toList()`:

```dart
..sort((b, a) => b.lastMessageInsertedAt.compareTo(a.lastMessageInsertedAt))
// ↑ ascending with confusingly named params
```

This is mathematically correct (ascending + reversed = descending) but misleading. A future developer might "fix" the naming and break the sort, or add `.reversed` when refactoring and accidentally produce ascending order.

See [investigation.md](../investigation.md) Finding 1.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [investigation.md](../investigation.md) Finding 1
- [ ] Confirm no other callers of `_sortChats()` exist: `rg "_sortChats" lib/ --type dart`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/chat/application/chat_cubit.dart` | modify | `_sortChats()` — lines 260-266 |

### Do NOT Modify

- `lib/core/services/chat_websocket_service.dart` — owned by task-02

## Implementation Steps

### Step 1: Replace `_sortChats()` with single-pass descending sort

Current (lines 260-266):
```dart
List<Chat> _sortChats(List<Chat> chats) {
  final sortedChats = List<Chat>.from(chats)
    ..sort(
      (b, a) => b.lastMessageInsertedAt.compareTo(a.lastMessageInsertedAt),
    );
  return sortedChats.reversed.toList();
}
```

Replace with:
```dart
List<Chat> _sortChats(List<Chat> chats) {
  return List<Chat>.from(chats)
    ..sort((a, b) => b.lastMessageInsertedAt.compareTo(a.lastMessageInsertedAt));
}
```

Note: `b.compareTo(a)` makes the sort descending (newest first). This is identical behavior to the original — just expressed clearly.

## Testing

- [ ] Sort output is identical to before (newest-first) — verify with a quick unit test if not already covered by task-04
- [ ] Existing chat cubit tests still pass: `fvm flutter test test/features/chat/`
- [ ] `fvm flutter analyze` clean
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — pure refactor with no behavior change

## Completion Criteria

- [ ] `_sortChats()` uses single-pass descending sort with clear parameter naming
- [ ] No `.reversed` call remains
- [ ] All existing tests pass
- [ ] Changes committed to `plan/SE-12760-chat-sort-ios/task-03-simplify-sort` branch
- [ ] Status updated in `status.md`
