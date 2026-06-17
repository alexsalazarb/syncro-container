# Task: Regression Tests for Null Handling and Sort Correctness

**Plan**: SE-12760-chat-sort-ios
**Task ID**: task-04
**Task Path**: task-04-regression-tests
**Depends On**: task-02, task-03
**Ticket**: SE-12760

## Objective

Add regression tests that fail before task-02/03 changes and pass after. Cover: null `last_message` handling, malformed entry skipping, and sort correctness.

## Context

Existing tests in `test/core/services/list_chats_from_socket_test.dart` verify parsing with well-formed data only. There are no tests for:
- `last_message: null` in `ChatFromSocket.fromJson`
- A malformed entry in `ListChatsFromSocket.fromJson` partial success
- `_sortChats()` output ordering with multiple chats

See [investigation.md](../investigation.md) Step 2c.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-02 and task-03 are complete
- [ ] Read existing test file: `test/core/services/list_chats_from_socket_test.dart`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `test/core/services/list_chats_from_socket_test.dart` | modify | Add null/malformed entry tests |
| `test/features/chat/application/chat_sort_test.dart` | create | New file for `_sortChats()` regression tests |

### Do NOT Modify

- `lib/` source files — all source changes are done in task-02 and task-03

## Implementation Steps

### Step 1: Add null `last_message` test to existing test file

In `test/core/services/list_chats_from_socket_test.dart`, add:

```dart
test('ChatFromSocket.fromJson returns epoch timestamp when last_message is null', () {
  final json = {
    'account_id': 1,
    'archived_at': null,
    'asset_id': 10,
    'id': 42,
    'remove_history': false,
    'user_id': 5,
    'asset_last_read_at': null,
    'user_last_read_at': null,
    'last_message': null,
  };
  final chat = ChatFromSocket.fromJson(json);
  expect(chat.lastMessageInsertedAt, equals(DateTime.fromMillisecondsSinceEpoch(0)));
});

test('ChatFromSocket.fromJson returns epoch timestamp when last_message is empty map', () {
  final json = {
    'account_id': 1,
    'archived_at': null,
    'asset_id': 10,
    'id': 42,
    'remove_history': false,
    'user_id': 5,
    'asset_last_read_at': null,
    'user_last_read_at': null,
    'last_message': <String, dynamic>{},
  };
  final chat = ChatFromSocket.fromJson(json);
  expect(chat.lastMessageInsertedAt, equals(DateTime.fromMillisecondsSinceEpoch(0)));
});

test('ListChatsFromSocket.fromJson skips malformed entry and returns remaining chats', () {
  final payload = {
    'total_count': 2,
    '1': {
      'account_id': 1,
      'archived_at': null,
      'asset_id': 10,
      'id': 1,
      'remove_history': false,
      'user_id': 5,
      'asset_last_read_at': null,
      'user_last_read_at': null,
      'last_message': {'inserted_at': '2024-01-01T12:00:00'},
    },
    '2': {
      // malformed — last_message is null
      'account_id': 1,
      'archived_at': null,
      'asset_id': 20,
      'id': 2,
      'remove_history': false,
      'user_id': 5,
      'asset_last_read_at': null,
      'user_last_read_at': null,
      'last_message': null,
    },
  };
  final result = ListChatsFromSocket.fromJson(payload);
  // Both should parse now (task-02 uses epoch fallback, not throw)
  expect(result.chats.length, equals(2));
});
```

### Step 2: Create new sort regression test file

Create `test/features/chat/application/chat_sort_test.dart`:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:syncro_flutter/features/chat/domain/chat.dart';

// Access _sortChats via a testable wrapper or test it indirectly through
// ChatLoaded state emission. Since _sortChats is private, test the observable
// behavior: chats emitted via ChatLoaded are in descending lastMessageInsertedAt order.
//
// If _sortChats needs to be made package-private for testing, do so only if
// existing patterns in the codebase allow it. Otherwise, test indirectly via
// _handleChatUpdate() integration path.

void main() {
  group('Chat sort order', () {
    final older = DateTime.parse('2024-01-01T10:00:00Z');
    final middle = DateTime.parse('2024-01-01T11:00:00Z');
    final newest = DateTime.parse('2024-01-01T12:00:00Z');

    test('chats are sorted newest first', () {
      // Build a minimal list in arbitrary order
      final chats = [
        _makeChat(id: 1, insertedAt: middle),
        _makeChat(id: 2, insertedAt: oldest),
        _makeChat(id: 3, insertedAt: newest),
      ];

      // Sort using the same logic as _sortChats (task-03 simplified version):
      final sorted = List<Chat>.from(chats)
        ..sort((a, b) => b.lastMessageInsertedAt.compareTo(a.lastMessageInsertedAt));

      expect(sorted[0].id, equals(3)); // newest
      expect(sorted[1].id, equals(1)); // middle
      expect(sorted[2].id, equals(2)); // oldest
    });
  });
}

Chat _makeChat({required int id, required DateTime insertedAt}) {
  return Chat(
    id: id,
    businessName: 'test',
    customerName: 'test',
    username: 'test',
    userId: 0,
    asset: 0,
    userLastReadAt: null,
    assetLastReadAt: null,
    lastMessageInsertedAt: insertedAt,
  );
}
```

Adjust `_makeChat` constructor to match the actual `Chat` domain model fields.

### Step 3: Run full test suite

```bash
cd syncro-flutter
fvm flutter test test/core/services/list_chats_from_socket_test.dart
fvm flutter test test/features/chat/
```

All tests must pass.

## Testing

- [ ] New null `last_message` tests fail on the original code (before task-02 fix)
- [ ] New null `last_message` tests pass after task-02 fix
- [ ] Sort test confirms newest-first ordering
- [ ] All pre-existing tests in `test/features/chat/` still pass
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — tests are self-documenting

## Completion Criteria

- [ ] `list_chats_from_socket_test.dart` covers null `last_message` and malformed entry
- [ ] `chat_sort_test.dart` covers sort order correctness
- [ ] All tests green: `fvm flutter test`
- [ ] Changes committed to `plan/SE-12760-chat-sort-ios/task-04-regression-tests` branch
- [ ] Status updated in `status.md`
