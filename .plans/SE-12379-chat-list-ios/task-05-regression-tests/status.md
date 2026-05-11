# Status: Add Regression Tests for All Four Fixes

**Current Status**: complete
**Last Updated**: 2026-05-11
**Agent**: —
**Branch**: plan/SE-12379-chat-list-ios/task-05-regression-tests
**PR**: N/A

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-05-11 | not-started | Task created |
| 2026-05-11 | in-progress | Writing regression tests |
| 2026-05-11 | complete | All tests written and passing |

## Blockers

None

## Artifacts

- `test/features/chat/infrastructure/list_chats_from_socket_test.dart` — Root Cause 1 regression (3 tests)
- `test/features/chat/application/new_chat_cubit_test.dart` — Root Cause 4 regression (2 tests)
- `test/features/chat/application/chat_cubit_refresh_test.dart` — Root Cause 2 regression (2 tests)

## Adaptations

- Root Cause 3 (tab-tap `startsWith` fix) not covered: requires a full widget test with navigation; out of scope for unit test regression.
- `ConnectivityService` is a hardcoded singleton with real I/O in its `initialize()` method; the `_connectivitySubscription` late field may not be initialized when `ChatCubit.close()` is called in tearDown. Handled with try/catch in tearDown so cleanup always completes.
- Second test in `chat_cubit_refresh_test.dart` uses `same(stateBefore)` instead of type assertion to be robust against `_initialize()` microtasks running between tests.
