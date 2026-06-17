# Status: Fix Chat List Not Refreshing on AppBar Back

**Current Status**: complete
**Last Updated**: 2026-06-17
**Agent**: Claude Code
**Branch**: plan/SE-12760-chat-sort-ios/task-06-fix-back-navigation-refresh
**PR**: N/A

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-06-17 | not-started | Task created — gap identified during SE-12760 investigation |
| 2026-06-17 | in-progress | Implementation started |
| 2026-06-17 | complete | Fix implemented, 60/60 tests passing, committed and pushed |

## What Was Done

Removed the `isNewChat` condition from `onBackPressed` in `chat_detail_page.dart:96`.

`leaveMessageChannel()` is now called unconditionally on AppBar back, consistent
with the `PopScope` path which already called it for all chats.

**File changed**: `lib/features/chat_detail/presentation/chat_detail_page.dart`

## Test Results

- `fvm flutter analyze lib/features/chat_detail/presentation/chat_detail_page.dart`: **No issues**
- `fvm flutter test test/features/chat/ test/features/chats/`: **60/60 passed**

## Blockers

None

## Artifacts

- Commit: `b2e86cb4`
- Branch: `plan/SE-12760-chat-sort-ios/task-06-fix-back-navigation-refresh`

## Adaptations

None — one-line fix, exactly as specified.
