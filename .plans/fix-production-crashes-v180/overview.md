# Plan: Fix Production Crashes — v1.8.0

**Status**: not-started
**Created**: 2026-09-07
**Last Updated**: 2026-09-07 (corrected — see Correction Log)
**Type**: Bug Fix (Type 3)
**Severity**: P2
**Ticket**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Trigger**: Internal Crashlytics audit requested by Alex, 2026-09-07 — full review of open iOS + Android issues before planning any fix
**Blast Radius**: ~3 users (Task 1) — low volume, non-fatal
**Master Plan**: None

## Bug Summary

One Crashlytics issue has a confirmed root cause with no existing plan: a null-safety gap in `Ticket.fromJson` (same family as the crashes fixed in `fix-production-crashes-v140`/`v152`). A second task performs housekeeping — closing 3 Crashlytics issues that are already fixed in code but still show as OPEN in the console because nobody closed them after the v152 fixes shipped.

## Correction Log

**2026-09-07**: This plan originally included a `task-02-webview-dispose-race` targeting Crashlytics issue `b0914a639eb5e4b996dec3e8f1147a4b`, hypothesizing the root cause was `login_web_view.dart`'s `dispose()` racing a pending auth challenge. While investigating branch sync before pushing, discovered `origin/main` already has commit `3f1afaf` ("plan(1.7.1-crashlytics): create fix plan for 8 open Crashlytics issues", 2026-08-28) — a plan not present on this branch (`POCAuth`) because it diverged from `main` before that commit. That plan's `task-05` targets the **same Crashlytics issue ID** with a different, better-evidenced root cause: `attachment_preview_view.dart:44` constructs a `WebViewController()` with **no `NavigationDelegate` at all** (confirmed via `rg` — zero matches for `NavigationDelegate`/`onHttpAuthRequest` in that file). That plan is `not-started`, already covers this issue plus 4 other items this plan had wrongly listed as "not yet investigated" (see Out of Scope below). `task-02` was removed here to avoid duplicating that work with an inferior root-cause hypothesis. Housekeeping task renumbered `task-03` → `task-02`.

## Root Cause

**Task 1 — `Ticket.fromJson` null cast**: `lib/features/ticket/ticket_home/domain/ticket.dart:54` casts `json['number'] as int` with no null-safety (line 53's `id` has the same gap). When the backend returns `number: null`, the cast throws inside a `compute()` isolate (`get_tickets_deserializer.dart:10`), failing the entire tickets-list parse for that page. Same defensive-cast pattern already applied to `CustomerInformation.fromJson` and `GetTicketsSettingsDeserializer` under `SE-11997` — just never applied here.

Full detail, stack traces, and confidence assessment in [investigation.md](investigation.md).

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| Ticket list (Android + iOS) | Parses `GET /tickets` response via `Ticket.fromJson` | Silent tickets-list load failure when `number`/`id` is null for any ticket in the page |
| Firebase Crashlytics console | Issue tracking | 3 stale OPEN issues creating false-positive noise for future triage |

## Scope

### In Scope
- `Ticket.fromJson` null-safe cast for `id`/`number` (Task 1)
- Regression test for Task 1
- Close 3 stale Crashlytics issues already fixed in code (Task 2)

### Out of Scope
- **Already covered by `.plans/1.7.1-crashlytics/` (on `main`, not-started, base branch `develop`)** — do not re-plan these, execute that plan instead:
  - iOS PlatformUtil EXC_BREAKPOINT regression (`ddf4c6780aa1a1d4452806dd8f5e381a`) — task-01 there
  - iOS Stack Overflow FATAL (`0bae269c5d2442a15a183ad4642da5c7`, top open iOS issue, 35 events/10 users) — task-02 there
  - iOS `NotificationManager.requestPermissions` firebase_messaging crash (`307055b68251cef77c20c6ca27b33816`, `4d17e7a500b300c48a84f037b2ded77a`) — task-03 there
  - `FirebirdSocketService` phoenix_socket channel errors (`aa0323e5c16c3e7546f84452e28eb8a3`, `be06f8e7c72127f022185820c3d9ad4b`) — task-04 there (NOT the same file as `ChatWebSocketService`, see below)
  - WebView auth-challenge in `AttachmentPreviewView` (`b0914a639eb5e4b996dec3e8f1147a4b`) — task-05 there
  - Android Play Core NPE cold-start (`5a34e9320c9c52a9d8dc5d1e21b24d49`) — task-06 there
- **Genuinely not yet planned by anyone** (real gaps, not this plan's scope):
  - `FragmentStateManager.createView` FATAL (Android, top open Android issue, 29 events/16 users) — root cause likely in Flutter's `FlutterActivity`/`FlutterFragment` embedding after the 3.44 SDK upgrade (`SE-12530`); needs its own embedding-focused investigation.
  - `ChatWebSocketService.startNewChat` "Failed to join channel" (Android `6cca598ad0eed80d3c1a344cb1d3af95`, iOS `f92cc8bc8266594c291fa984b0c75366`) — distinct file from `FirebirdSocketService` (which `1.7.1-crashlytics/task-04` covers) and distinct from the already-fixed `.connect` null-check; channel-join lifecycle in the chat service specifically. Not investigated by anyone yet.
  - Low-volume native/OS crashes not in either plan (missing `libflutter.so`, ANRs, `SuperNotCalledException` from `flutter_inappwebview`, `RemoteServiceException`) — third-party/OS-level, low ROI, monitor only.

## Kill Criteria

- Fix introduces worse behavior than the original bug (e.g. silently dropping tickets instead of showing them with a placeholder)
- Root cause is disproved by new evidence once implementation starts

## Task Summary

| Task Path | Title | Status | Priority | Depends On |
|-----------|-------|--------|----------|------------|
| task-01-ticket-fromjson-nullsafety | Fix `Ticket.fromJson` null cast on `id`/`number` | not-started | HIGH — same pattern as 4 prior confirmed fixes | — |
| task-02-crashlytics-housekeeping | Close 3 stale Crashlytics issues already fixed in code | not-started | LOW — administrative, no code change | — |

Both tasks are independent and can run in parallel.

## Branch Convention

Task branches: `plan/fix-production-crashes-v180/{task-path}`
Merge target: `main`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `lib/features/ticket/ticket_home/domain/ticket.dart` | **Primary fix** — task-01, lines 52-54 |
| `lib/features/ticket/ticket_home/infrastructure/get_tickets_deserializer.dart` | Context — where the isolate-level failure surfaces (line 10) |
| `test/features/ticket/ticket_home/domain/ticket_test.dart` | task-01 regression test — extend existing suite |

## Crashlytics Issue IDs

| Issue | Platform | ID | Type | Task |
|-------|----------|----|----|------|
| `Ticket.fromJson` null cast | Android | `f91fb1b40d29c211bdf65655b75937b8` | NON_FATAL | task-01 |
| `Ticket.fromJson` null cast | iOS | `50306dfd3ee04e4d8c71ae4d212954dc` | NON_FATAL | task-01 |
| `asset_filter_deserializer` (stale, fixed in v152) | Android | `9a1d349f325abb563d2b26653a1b993c` | NON_FATAL | task-02 |
| `WorksheetTemplateDeserializer` (stale, fixed in v152) | Android | `de94f83f3e5808649141d2c1d40c58a8` | NON_FATAL | task-02 |
| `DioMixin` 504 (stale, backend-owned, out of scope) | Android | `7ec1fff22d998a861ff0d1705d05518d` | NON_FATAL | task-02 |

## Success Criteria

- [ ] `Ticket.fromJson` no longer throws when `id` or `number` is null in the API response
- [ ] Regression test fails before the fix and passes after
- [ ] All existing tests continue to pass
- [ ] `flutter analyze` passes with no new warnings
- [ ] 3 stale Crashlytics issues marked CLOSED
- [ ] 0 new occurrences of the fixed issue in Crashlytics after release

## Defects

<!-- Bugs discovered during plan execution. Added by execute-task's Bug Discovery Protocol (Step 5a). -->

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

- [ ] All tasks complete or adapted
- [ ] Bug no longer reproducible with original repro steps
- [ ] Regression test: red before fix, green after (verified)
- [ ] All existing tests pass
- [ ] investigation.md root cause matches the actual fix (no drift)
- [ ] KB/documentation updated or explicitly marked not needed
- [ ] Staging verification complete

## Revert Plan

**Revert trigger**: New Crashlytics events for the fixed issue within 48h of release, or a new regression in ticket-list loading
**Revert steps**: Revert the merge commit for the affected task's branch; re-open the corresponding Crashlytics issue(s)
**Rollback owner**: Alex Salazar

## References

- **Ticket**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: [fix-production-crashes-v140](../fix-production-crashes-v140/overview.md) (SE-11997, same null-safety pattern), [fix-production-crashes-v152](../completed/fix-production-crashes-v152/overview.md) (asset_filter/worksheet issues being closed in task-02), [1.7.1-crashlytics](../1.7.1-crashlytics/overview.md) (covers 5 of the issues originally miscategorized as out-of-scope/uninvestigated in this plan — execute that plan for those, not this one)
