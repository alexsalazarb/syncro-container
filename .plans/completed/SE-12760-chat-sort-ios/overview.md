# Plan: iOS Chat List Not Sorted by Most Recent

**Status**: complete
**Created**: 2026-06-16
**Last Updated**: 2026-06-17
**Completed**: 2026-06-17
**Type**: Bug Fix (Type 3)
**Severity**: P2 — Chat list order is wrong on iOS for specific accounts; chats are visible but misordered. Workaround: logout/login temporarily restores correct order.
**Ticket**: SE-12760
**Assigned Dev**: Alex Salazar
**Assigned QA**: unassigned
**Trigger**: Support ticket — customer SA 75758 (hkge.syncromsp.com), spt-elevated label
**Blast Radius**: iOS users on any account (race condition is universal); visible as flicker + potential misordering
**Master Plan**: None

## Bug Summary

On iOS, the Chats tab displays conversations in the wrong order — not sorted by most recent message. A customer video reveals the list appears in one order and **immediately reorders** (visible flicker), and the final order is still incorrect. The web app shows the correct order.

## Root Cause

**Confidence: High** — Confirmed via diagnostic logging session (task-01 adapted).

**Primary (mobile)**: `ChatWebSocketService._createListeners()` subscribes to `_socket.messageStream` and `_channel!.messages` without storing or canceling previous `StreamSubscription` objects. Due to a race condition between the `unawaited(chatWebSocketService.init())` fire-and-forget in `route_cubit.dart:97` and `ChatCubit.getChats()` → `getInteractions()` reinit fallback, `_createListeners()` is called twice. This accumulates duplicate listeners. Every `interactions_batched` event fires both listeners → two `_handleChatUpdate()` calls with concurrent async operations → different `ChatLoaded` emissions in rapid succession → visible flicker.

**Secondary (possible server-side)**: `last_message.inserted_at` values in the socket payload may not match the "last activity" order the web app uses for certain accounts. This manifests as wrong final sort order even after the race condition is fixed. Confirm after task-05.

A third fragility: `ChatFromSocket.fromJson` performs an unsafe cast on `last_message` — if null, the entire list parse crashes silently. This is a latent risk independent of the ordering bug.

See [investigation.md](investigation.md) for full root cause analysis.

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| `ChatWebSocketService._createListeners()` | Attaches socket/channel listeners | **Root cause** — accumulates duplicate subscriptions |
| `ChatWebSocketService` | Parses `interactions_batched` payload | `ChatFromSocket.fromJson` fragile on null `last_message` |
| `ChatCubit._sortChats()` | Sorts chats by `lastMessageInsertedAt` | Confusing double-reversal pattern; correct but misleading |
| `ChatCubit._handleChatUpdate()` | Receives WebSocket push and emits sorted state | Fires twice due to duplicate listeners |

## Scope

### In Scope
- Fix duplicate listener accumulation in `_createListeners()` (task-05 — primary fix)
- Resilient null `last_message` handling in `ChatFromSocket.fromJson` (task-02)
- Simplify `_sortChats()` double-reversal to a clear single-pass sort (task-03)
- Regression tests for null handling and sort correctness (task-04)

### Out of Scope
- Server-side fix for `interactions_batched` sort order — out of mobile scope; file a backend ticket if final order is still wrong after task-05
- WebSocket reconnection logic — unrelated
- Android platform — race condition exists but iOS users more affected

## Kill Criteria

- Fix introduces a regression in chat list loading on Android or iOS
- Duplicate listener fix causes `!_joinedOnce` Phoenix assertion errors on hot restart

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-diagnostic-logging | Diagnostic logging — capture raw socket payload | adapted | — |
| task-02-fix-null-last-message | Resilient fromJson for null last_message | complete | — |
| task-03-simplify-sort | Simplify _sortChats double-reversal | complete | — |
| task-04-regression-tests | Regression tests for null handling and sort | complete | task-02, task-03 |
| task-05-fix-duplicate-listeners | Fix duplicate listeners in _createListeners() | complete | — |
| task-06-fix-back-navigation-refresh | Fix chat list not refreshing on AppBar back from chat detail | complete | — |

## Branch Convention

Task branches: `plan/SE-12760-chat-sort-ios/{task-path}`
Merge target: `main`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `lib/core/services/chat_websocket_service.dart:394` | **task-05** — `_createListeners()` — add subscription fields + cancel before re-subscribe |
| `lib/core/services/chat_websocket_service.dart:1163` | **task-02** — `ChatFromSocket.fromJson` null `last_message` unsafe cast |
| `lib/features/chat/application/chat_cubit.dart:267` | **task-03** — `_sortChats()` double-reversal cleanup |
| `lib/features/chat/application/chat_cubit.dart:181` | `_handleChatUpdate()` — fires twice due to duplicate listeners (fixed by task-05) |
| `test/features/chat/application/` | **task-04** — existing test directory |
| `test/core/services/list_chats_from_socket_test.dart` | **task-04** — existing WebSocket parsing tests |

## Success Criteria

- [ ] Chat list loads without visible flicker on navigation to Chats tab
- [ ] `ChatFromSocket.fromJson` handles null `last_message` without throwing
- [ ] `_sortChats()` uses a single-pass descending sort (behavior unchanged)
- [ ] Regression test fails before task-02/03 changes, passes after
- [ ] All existing tests pass
- [ ] Required KB / documentation updates complete or explicitly marked not needed

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

- [ ] All tasks complete or adapted
- [ ] No visible flicker on navigation to Chats tab
- [ ] Bug no longer reproducible with original repro steps
- [ ] Regression test: red before fix, green after (verified)
- [ ] All existing tests pass
- [ ] investigation.md root cause matches the actual fix (no drift)
- [ ] All consumers handle the changed behavior gracefully
- [ ] KB/documentation updated or explicitly marked not needed
- [ ] Ticket transitioned (or transition noted for manual action)
- [ ] Staging verification complete

## Revert Plan

**Revert trigger**: Chat list stops showing chats on iOS or Android
**Revert steps**: `git revert` task-05 commit; `git revert` task-02 commit (null guard); task-03 is cosmetic with no behavior change
**Rollback owner**: On-call mobile engineer

## References

- **Ticket**: https://syncrotech.atlassian.net/browse/SE-12760
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: [SE-12379-chat-list-ios](../SE-12379-chat-list-ios/overview.md) — original iOS chat fix that introduced `_sortChats()` + `lastMessageInsertedAt` parsing
