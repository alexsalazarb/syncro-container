# Task: Add Regression Tests for All Four Fixes

**Plan**: SE-12379-chat-list-ios
**Task ID**: task-05
**Task Path**: task-05-regression-tests
**Depends On**: task-01-fix-websocket-parsing, task-02-fix-new-chat-refresh, task-03-fix-tab-tap-refresh, task-04-fix-empty-state-timeout
**Ticket**: SE-12379

## Objective

Write regression tests for each of the four root causes fixed in tasks 01–04. Each test must fail on the original code and pass after the fix.

## Context

See [investigation.md](../investigation.md) for all root causes.

Existing `ChatCubit` and `ChatWebSocketService` tests exist in `test/features/chat/`. Follow the existing test patterns (mockito, bloc_test, fakes). Regenerate mocks if new injectable dependencies were added in tasks 01 or 02.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify tasks 01–04 are all complete before writing tests
- [ ] Read [investigation.md](../investigation.md) for all four root causes
- [ ] Check `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Browse `test/features/chat/` to understand existing test conventions
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `test/features/chat/chat_websocket_service_test.dart` | create or modify | Regression for Root Cause 1 |
| `test/features/chat/application/new_chat_cubit_test.dart` | create or modify | Regression for Root Cause 4 |
| `test/features/chat/application/chat_cubit_refresh_test.dart` | create or modify | Regression for Root Cause 2 |

### Do NOT Modify

- Any production source files — this task is tests only

## Implementation Steps

### Test 1: ListChatsFromSocket.fromJson with mixed payload (Root Cause 1)

File: `test/features/chat/chat_websocket_service_test.dart` (or equivalent)

```dart
test('fromJson skips non-Map values and returns only chat entries', () {
  final payload = {
    '123': {
      'id': 123,
      'account_id': 1,
      // ... minimal ChatFromSocket fields
    },
    'total_count': 5,  // ← integer — was causing TypeError
    '456': {
      'id': 456,
      'account_id': 1,
      // ...
    },
  };

  final result = ListChatsFromSocket.fromJson(payload);

  expect(result.chats.length, 2);  // skips total_count
  expect(result.chats.map((c) => c.id), containsAll([123, 456]));
});
```

Confirm the minimum required fields for `ChatFromSocket.fromJson` by reading the actual `fromJson` implementation before writing the test.

### Test 2: NewChatCubit calls ChatCubit.refresh after creation (Root Cause 4)

File: `test/features/chat/application/new_chat_cubit_test.dart`

Create a mock `ChatCubit` and verify `refresh(showLoading: false)` is called after the `newChatSocketStream` emits a new chat:

```dart
test('calls ChatCubit.refresh after new chat is emitted', () async {
  final mockChatCubit = MockChatCubit();
  final service = FakeChatWebSocketService();
  final cubit = NewChatCubit(chatCubit: mockChatCubit, service: service);

  service.emitNewChat(...);

  verify(mockChatCubit.refresh(showLoading: false)).called(1);
});
```

Adapt the constructor signature to match the actual implementation from task-02.

### Test 3: refresh(showLoading: true) emits ChatLoading, not ChatLoaded([]) (Root Cause 2)

File: `test/features/chat/application/chat_cubit_refresh_test.dart` (or add to existing cubit test)

```dart
blocTest<ChatCubit, ChatState>(
  'refresh(showLoading: true) emits ChatLoading, not ChatLoaded([])',
  build: () => chatCubit,
  act: (cubit) => cubit.refresh(showLoading: true),
  expect: () => [isA<ChatLoading>(), /* followed by whatever the WS returns */],
);
```

Also verify the timer fires `ChatError` when state is `ChatLoaded(chats: [])`:

```dart
blocTest<ChatCubit, ChatState>(
  'timeout fires ChatError when state is ChatLoaded with empty list',
  ...
);
```

### Step 4: Regenerate mocks if needed

If task-02 added `ChatCubit` as a constructor parameter to `NewChatCubit`, regenerate mocks:

```bash
cd syncro-flutter
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

### Step 5: Run all tests

```bash
cd syncro-flutter
fvm flutter test test/features/chat/
fvm flutter analyze
```

All tests must pass.

## Testing

- [ ] `ListChatsFromSocket.fromJson` test: fails on original code (TypeError), passes after task-01 fix
- [ ] `NewChatCubit` refresh test: fails on original code (no call), passes after task-02 fix
- [ ] `ChatCubit.refresh()` loading state test: fails on original code (`ChatLoaded` emitted), passes after task-04 fix
- [ ] All other chat tests still pass
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required.

## Completion Criteria

- [ ] Regression tests written for all four root causes
- [ ] Each test was verified to fail before the corresponding fix and pass after
- [ ] `fvm flutter test test/features/chat/` — all green
- [ ] `fvm flutter analyze` clean
- [ ] Changes committed to `plan/SE-12379-chat-list-ios/task-05-regression-tests` branch
- [ ] Status updated in `status.md`
