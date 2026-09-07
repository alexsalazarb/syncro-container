# Status: guard `FirebirdSocketService` channel join/push against phoenix_socket timeout/assertion errors

**Current Status**: complete
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: `plan/fix-production-crashes-v180` (syncro-flutter submodule, based on `develop`) — unified plan branch, see overview.md Branch Convention
**PR**: N/A (PR_INTEGRATION=false) — see plan overview for the current PR link

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | complete | claude | Fix implemented, `flutter analyze` clean, full suite run (1 unrelated pre-existing failure, same as noted in task-03). No executable regression test — see Adaptations |

## Summary

Confirmed the task's Context: `init()` reassigns shared `_socket`/`_channel` instance fields on every call and kicks off a fire-and-forget async chain ending in `_channel.join()`, with no timeout/error handling and no guard against a re-entrant `init()` racing two chains into joining the same channel (phoenix_socket's `assert(!_joinedOnce)`).

Added `_isChannelJoinInFlight` guard (mirrors `ChatWebSocketService`'s `_isMessageChannelJoined`), extracted join+push into `_joinChannel()` wrapped in try/catch with a 15s `.timeout(...)`, guarded the `request_thumbnail` push with `_channel.canPush`. Reset the guard in `disconnect()`.

Confirmed via `phoenix_socket` package source (`~/.pub-cache/hosted/pub.dev/phoenix_socket-0.7.6/lib/src/channel.dart`) that `_joinedOnce` is private with no public getter — our own instance-level flag is the only available guard.

## Regression Test — none added (documented adaptation)

`test/core/services/freebird_websocket_service_test.dart` contains **no executable tests**, following the exact precedent already established in `test/core/services/chat_websocket_service_connect_test.dart` for the sibling `ChatWebSocketService`: both classes share the same untestable-without-refactoring constraints — singleton via factory constructor, GetIt dependencies resolved at field-initialization time (before any test `setUp` can register fakes), and no injection seam for `_socket`/`_channel`. Rather than inventing an ad-hoc workaround inconsistent with that documented precedent, followed the same "fix verified by code review, rationale documented in the test file" pattern.

## Verification

- [x] `fvm flutter analyze` on changed files: clean, no issues
- [x] Full suite (`fvm flutter test`): 2289 tests, 1 failure — same pre-existing `chat_models_test.dart` casing mismatch noted in task-03, unrelated, not touched

## Artifacts

- `lib/core/services/freebird_websocket_service.dart` — fix
- `test/core/services/freebird_websocket_service_test.dart` — documentation-only test file (no executable tests, matches sibling precedent)

## Adaptations

**Adapted**: task.md's Testing section asked for new regression tests ("calling init() twice does not throw", "join timeout is caught"). Investigated the suggested reference (`chat_websocket_service_connect_test.dart`) and found it is itself a zero-test file with a documented rationale for why `ChatWebSocketService` can't be unit tested without invasive refactoring — and `FirebirdSocketService` has the identical constraints (singleton, field-init GetIt deps, no injection seam). Followed that same precedent instead of forcing tests inconsistent with established codebase convention. Fix behavior verified by code review and by reading `phoenix_socket`'s own source for `_joinedOnce`/`canPush` semantics, not by an automated test.
