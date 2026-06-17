# Status: Resilient fromJson for Null last_message

**Current Status**: complete
**Last Updated**: 2026-06-17
**Agent**: Claude Code
**Branch**: plan/SE-12760-chat-sort-ios/task-02-fix-null-last-message
**PR**: N/A

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-06-16 | not-started | Task created |
| 2026-06-17 | in-progress | Implementation started |
| 2026-06-17 | complete | Fix implemented, 60/60 tests passing, committed and pushed |

## What Was Done

Added `_parseLastMessageInsertedAt` static helper to `ChatFromSocket` that
returns `DateTime.fromMillisecondsSinceEpoch(0)` when `last_message` is null
or `inserted_at` is missing. Replaced unsafe cast in `fromJson` with the helper.

Added per-entry `try/catch` in `ListChatsFromSocket.fromJson` using `.expand()`
so a single malformed entry is silently skipped instead of crashing the entire list.

**Adaptation**: `debugPrint` logging in the catch block was omitted to avoid
re-adding the `flutter/foundation.dart` import that was removed with diagnostic
logging. Silent skip is the correct production behavior.

## Test Results

- `fvm flutter analyze lib/core/services/chat_websocket_service.dart`: **No issues**
- `fvm flutter test test/features/chat/ test/features/chats/ test/core/services/`: **60/60 passed**

## Blockers

None

## Artifacts

- Commit: `4c5312af`
- Branch: `plan/SE-12760-chat-sort-ios/task-02-fix-null-last-message`

## Adaptations

Omitted `debugPrint` in catch block — `flutter/foundation.dart` import was
previously cleaned up with diagnostic logging removal. Silent skip is correct
for a production model parser.
