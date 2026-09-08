# Status: ChatWebSocketService double-join race ('!_joinedOnce')

**Current Status**: complete
**Last Updated**: 2026-09-08
**Agent**: claude
**Branch**: N/A — fixed directly on `develop` in the `syncro-flutter` submodule (not on the `plan/fix-production-crashes-v180` branch, which was already merged and closed before this defect was found)
**PR**: N/A (PR_INTEGRATION=false)

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-09-08 | complete | claude | Found and fixed in the same session, ad-hoc — not picked up from a pre-existing not-started task. Task created retroactively to correct this plan's own stale "already hardened / out of scope" claim about `chat_websocket_service.dart` and to formally close the "not investigated by anyone yet" item already logged in overview.md. |

## Summary

User reported a live log trace showing `PhoenixChannel.join()`'s `assert(!_joinedOnce)` firing in `ChatWebSocketService._tryInitialConnection`, more often since the v1.8.0 Firebase Stability changes. Confirmed via `phoenix_socket` source (`~/.pub-cache/hosted/pub.dev/phoenix_socket-0.7.7/lib/src/socket.dart:372-393`) that `PhoenixSocket.addChannel()` caches channels by topic and returns the same instance on repeat calls — so any two concurrent callers reaching `.join()` on the same topic crash on the second call.

Found the race was NOT limited to `_tryInitialConnection()` (user channel) — the identical unguarded pattern existed in `_createAndJoinChannel()` (message channel), which is what `startNewChat()` calls. That is exactly the Crashlytics issue this plan already listed under "Genuinely not yet planned by anyone" (Android `6cca598ad0eed80d3c1a344cb1d3af95`, iOS `f92cc8bc8266594c291fa984b0c75366`) — this task resolves it.

Fixed both:
- `_tryInitialConnection()`: added `Completer<void>? _initializingCompleter` — single-flight guard, safe because all callers target the same fixed user-channel topic.
- `_createAndJoinChannel()`: added `Future<void>? _channelOperationLock` — a turnstile, not single-flight, because each call can target a different asset topic (sharing a cached result would tell a waiter an unrelated topic is ready). Added a `_needsNewChannel(topic)` re-check after acquiring the lock to skip redundant rejoin.
- Also wrapped the 3 `onDone` reconnection calls in `runZonedGuarded`, matching the pattern already used elsewhere in the file (`init()`, connectivity listener, health-check timer).

## Regression Test — none added (documented adaptation, same precedent as task-04)

Same untestable-without-refactoring constraints as `task-04-freebird-socket-channel-errors` and the existing `chat_websocket_service_connect_test.dart` file: singleton via factory constructor, `GetIt` dependencies resolved at field-initialization time, no injection seam for `_socket`/`_channel`/`_messageChannel` to deterministically simulate two racing callers in a unit test. Followed the same "fix verified by code review + reading the actual library source + full suite passing clean" pattern already established for this class.

## Verification

- [x] `fvm dart analyze lib/core/services/chat_websocket_service.dart`: no issues found
- [x] `fvm flutter test test/core/services/chat_websocket_service_connect_test.dart test/features/chat/`: 56 tests passed, 0 regressions
- [ ] Manual staging verification (double-tap send, airplane-mode toggle while chat screen open) — recommended, not yet performed

## Artifacts

- `syncro-flutter/lib/core/services/chat_websocket_service.dart` — fix (commit pending in `syncro-flutter`, referencing `SE-13805 / fix-production-crashes-v180 task-09`)

## Adaptations

**Adapted**: this task was created after the fix was already implemented, in response to a user-reported log rather than from a pre-planned Testing section. No new task.md Testing checklist existed to deviate from; the adaptation is the same one task-04 already established as precedent (no executable regression test, code-review + library-source verification instead) — see Regression Test section above.

**Plan correction**: this task also corrects two stale claims in `overview.md` (Out of Scope line 63, Key Files line 116) that asserted `chat_websocket_service.dart` was already fully guarded — that verification, done during original planning, missed `_tryInitialConnection()` and `_createAndJoinChannel()`.
