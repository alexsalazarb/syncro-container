# Status: Regression Tests for Null Handling and Sort Correctness

**Current Status**: complete
**Last Updated**: 2026-06-17
**Agent**: Claude Code
**Branch**: plan/SE-12760-chat-sort-ios/task-04-regression-tests
**PR**: N/A

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-06-16 | not-started | Task created |
| 2026-06-17 | in-progress | Implementation started |
| 2026-06-17 | complete | Tests written and passing, committed and pushed |

## What Was Done

**Modified**: `test/features/chat/infrastructure/list_chats_from_socket_test.dart`
- Added `ChatFromSocket.fromJson — null last_message handling` group (3 tests):
  - returns epoch when `last_message` is null
  - returns epoch when `last_message` has no `inserted_at`
  - parses valid `inserted_at` correctly
- Added `ListChatsFromSocket.fromJson — malformed entry handling` group (2 tests):
  - skips entries with missing required fields
  - returns all chats when entries have null `last_message` (epoch fallback)

**Created**: `test/features/chat/application/chat_sort_test.dart`
- `_sortChats — newest first ordering` group (4 tests):
  - three chats sorted in descending order
  - single-element list unchanged
  - same-timestamp chats preserve list length
  - epoch-fallback chats sort to the bottom

**Branch note**: task-02 and task-03 commits were cherry-picked onto this branch
so the test suite passes. The tests would fail on the pre-fix code.

## Test Results

- `fvm flutter test test/features/chat/infrastructure/list_chats_from_socket_test.dart test/features/chat/application/chat_sort_test.dart`: **12/12 passed**
- `fvm flutter test test/features/chat/ test/features/chats/`: **69/69 passed**

## Blockers

None

## Artifacts

- Commit: `c46653a9`
- Branch: `plan/SE-12760-chat-sort-ios/task-04-regression-tests`

## Adaptations

Cherry-picked task-02 (`4c5312af`) and task-03 (`3af486ec`) commits onto this
branch so the regression tests pass. When integrating into develop, task-02 and
task-03 changes must be applied before task-04.
