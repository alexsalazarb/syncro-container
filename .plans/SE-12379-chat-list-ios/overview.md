# Plan: iOS Chat List Not Showing Initiated or Existing Chats

**Status**: not-started
**Created**: 2026-05-11
**Last Updated**: 2026-05-11
**Type**: Bug Fix (Type 3)
**Severity**: P1 — Core feature (chat list) broken for all iOS users; no workaround
**Ticket**: SE-12379
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Trigger**: Support ticket — Sev-2 (spt-elevated label)
**Blast Radius**: All iOS users — cannot see any chats (existing or newly created)
**Master Plan**: None

## Bug Summary

On iOS, the Chats tab shows "No chats yet" even when the user has active chats. Newly created chats also fail to appear after returning to the chat list. The issue is iOS-specific and not reproducible on Android.

## Root Cause

Four compounding issues are identified. The primary root cause is an unsafe cast in `ListChatsFromSocket.fromJson` (`chat_websocket_service.dart:1113`): `json.values` is mapped with `value as Map<String, dynamic>` without a type guard. When the WebSocket payload contains non-Map values (e.g., a `total_count` integer), a `TypeError` is thrown silently — the entire WebSocket update is dropped and the chat list never populates. The three secondary causes compound the issue: `refresh(showLoading: true)` emits an empty `ChatLoaded([])` before the WebSocket responds and the safety-net timer doesn't cover the `ChatLoaded` state; the tab-tap refresh guard checks exact route equality (`/chat`) which fails after returning from chat detail (`/chat-detail`); and `NewChatCubit` never triggers `ChatCubit.refresh()` after a chat is created.

See [investigation.md](investigation.md) for full root cause analysis.

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| `ChatWebSocketService` | Parses WebSocket `interactions_batched` payload | Primary — silent crash drops all chat data |
| `ChatCubit` | State management for chat list | Secondary — empty state shown permanently after refresh |
| `HomePage` (tab-tap handler) | Triggers refresh on tab tap | Tertiary — refresh not triggered after chat detail return |
| `NewChatCubit` | Creates new chats | Quaternary — no explicit refresh after chat creation |

## Scope

### In Scope
- Type-guard fix in `ListChatsFromSocket.fromJson`
- Trigger `ChatCubit.refresh()` from `NewChatCubit` after successful chat creation
- Fix tab-tap refresh guard for the chats branch
- Fix `refresh(showLoading: true)` empty state + timeout coverage
- Regression tests for all four fixes

### Out of Scope
- WebSocket reconnection logic — unrelated to this bug
- Chat detail page behavior — not affected
- Android platform — not affected by this bug

## Kill Criteria

- Fix introduces a regression in chat list loading on Android
- Root cause 1 is disproved (payload never contains non-Map values) — demote to lower priority

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-fix-websocket-parsing | Fix unsafe cast in ListChatsFromSocket.fromJson | not-started | — |
| task-02-fix-new-chat-refresh | Refresh chat list from NewChatCubit after creation | not-started | — |
| task-03-fix-tab-tap-refresh | Fix tab-tap refresh guard after chat detail return | not-started | — |
| task-04-fix-empty-state-timeout | Fix empty state flash and timeout in refresh() | not-started | — |
| task-05-regression-tests | Add regression tests for all four fixes | not-started | task-01, task-02, task-03, task-04 |

## Branch Convention

Task branches: `plan/SE-12379-chat-list-ios/{task-path}`
Merge target: `main`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `lib/core/services/chat_websocket_service.dart:1111` | **Primary fix** — `ListChatsFromSocket.fromJson` unsafe cast |
| `lib/features/chat/application/new_chat_cubit.dart` | **Task-02** — missing `ChatCubit.refresh()` call |
| `lib/features/home/presentation/home_page.dart:133` | **Task-03** — tab-tap guard route equality check |
| `lib/features/chat/application/chat_cubit.dart:342` | **Task-04** — empty state emitted before WebSocket responds |

## Success Criteria

- [ ] iOS chat list shows existing chats on app launch
- [ ] Newly created chats appear in the list after returning to Chats tab
- [ ] Tapping the Chats tab after returning from a chat detail triggers a refresh
- [ ] Regression test fails before fix, passes after
- [ ] All existing tests pass
- [ ] Required KB / documentation updates complete or explicitly marked not needed

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

- [ ] All tasks complete or adapted
- [ ] Bug no longer reproducible with original repro steps
- [ ] Regression test: red before fix, green after (verified)
- [ ] All existing tests pass
- [ ] investigation.md root cause matches the actual fix (no drift)
- [ ] All consumers handle the changed behavior (if applicable)
- [ ] KB/documentation updated or explicitly marked not needed
- [ ] Ticket transitioned (or transition noted for manual action)
- [ ] Staging verification complete

## Revert Plan

**Revert trigger**: Chat list stops loading on Android, or crash rate increases
**Revert steps**: `git revert` the task-01 commit; other tasks are additive and low risk
**Rollback owner**: On-call mobile engineer

## References

- **Ticket**: https://syncrotech.atlassian.net/browse/SE-12379
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: None
