# Plan: Fix Production Crashes — v1.8.0

**Status**: not-started
**Created**: 2026-09-07 (tasks 01-06 originally created 2026-08-28 as a separate plan, merged in — see Merge Log)
**Last Updated**: 2026-09-07
**Type**: Bug Fix (Type 3)
**Severity**: P1 (task-01 is a confirmed FATAL regression already reproduced in production; other tasks are individually P2/P3 — see Task Summary)
**Ticket**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
**Base Branch**: `develop` — confirmed via `git branch --show-current` in `syncro-flutter` and `git log main..develop`: `develop` is ahead of `main` (has the 1.8.0 version bump, the merged passkey-login work, and Patrol E2E coverage) — `develop` is the real integration branch, not `main`
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Trigger**: Internal Crashlytics audit requested by Alex, 2026-09-07 (tasks 07-08), plus a live 7-day Crashlytics report following the 1.7.1 release, 2026-08-28 (tasks 01-06)
**Blast Radius**: ~30 users combined across iOS + Android (see per-task Crashlytics numbers below) — task-01 is FATAL and already reproduced in the field; the rest are lower-volume FATAL/NON_FATAL
**Master Plan**: None

## Bug Summary

Nine Crashlytics issues across 8 root-cause clusters, covering the full open-issue backlog on both platforms (`com.servably.syncro.mobile`) plus 3 stale issues that just need closing. Consolidates two plans created 10 days apart into one: `1.7.1-crashlytics` (created 2026-08-28, covered 8 issues from the post-1.7.1 field report) and `fix-production-crashes-v180` (created 2026-09-07, covered a `Ticket.fromJson` null-safety gap + housekeeping). See Merge Log below for why they were separate and why they're now one.

## Merge Log

**2026-09-07**: This plan absorbed `.plans/1.7.1-crashlytics/` (6 tasks, phases 1-2) after the user asked whether the two plans could be combined. They existed separately only because `fix-production-crashes-v180` was authored on a branch (`POCAuth`) that had diverged from `main` before `1.7.1-crashlytics` was committed there — see the Correction Log entry below for how that was discovered. No file or scope overlap between the two beyond what the Correction Log already resolved (the WebView issue). Renumbering: the original `1.7.1-crashlytics` tasks kept their `task-01` through `task-06` IDs and phase structure; this plan's own 2 tasks were renumbered `task-01`→`task-07` and `task-02`→`task-08`, both placed in Phase 2 (independent, parallelizable, no shared files with anything else in the plan). Base branch corrected from `main` (this plan's original, wrong assumption) to `develop` (confirmed correct by the absorbed plan, verified again during the merge via `git log main..develop`).

## Correction Log

**2026-09-07** (predates the merge above): This plan originally included a `task-02-webview-dispose-race` targeting Crashlytics issue `b0914a639eb5e4b996dec3e8f1147a4b`, hypothesizing the root cause was `login_web_view.dart`'s `dispose()` racing a pending auth challenge. While investigating branch sync before pushing, discovered `origin/main` already had `1.7.1-crashlytics`'s `task-05`, targeting the **same Crashlytics issue ID** with a different, better-evidenced root cause: `attachment_preview_view.dart:44` constructs a `WebViewController()` with **no `NavigationDelegate` at all** (confirmed via `rg` — zero matches for `NavigationDelegate`/`onHttpAuthRequest` in that file). The retracted analysis is kept in [investigation.md](investigation.md) as a documented methodology lesson. That issue is now `task-05` in this merged plan (see Task Summary) — the correct root cause is already in `phase-2/task-05-webview-attachment-auth-challenge/task.md`.

## Root Cause Summary

Full detail for tasks 01-06 lives in each task's own `task.md` Context section (written during the original `1.7.1-crashlytics` planning session). Full detail for tasks 07-08 is in [investigation.md](investigation.md).

| Task | Issue | Root Cause (one-liner) |
|------|-------|------------------------|
| task-01 | `PlatformUtil.swift` EXC_BREAKPOINT | **Confirmed & escalated 2026-09-07**: `flutter_inappwebview_ios` 1.1.2 (transitive via `oauth_webauth`), force-unwrap of `plugin.registrar!` at `PlatformUtil.swift:15`. Matches unresolved upstream issue [#2368](https://github.com/pichillilorenzo/flutter_inappwebview/issues/2368) — no released fix exists. No code change applied per Kill Criteria; see task's `status.md` for full findings |
| task-02 | Stack Overflow (FlutterError) | **BLOCKED 2026-09-07**: confirmed genuine infinite recursion (identical repeating offset cycle across both variants), but no Dart symbolication available to identify the function — ruled out `chat_websocket_service.dart` and the `ticket_attachment` screens as candidates. 0 events on 1.7.1/1.8.0 (dormant ~2.5 months). See task's `investigation.md` |
| task-03 | `NotificationManager.requestPermissions` crash | **Fixed 2026-09-07**: wrapped `messaging.requestPermission(...)` in try/catch, returns `false` on failure. Regression test verified red-before/green-after. Branch: `plan/fix-production-crashes-v180/phase-2/task-03-notification-permission-crash`, pushed to `bla` (per user request), PR creation link in task's `status.md` |
| task-04 | `FirebirdSocketService` phoenix_socket errors | Unguarded channel join/push (`_channel.join().future.then(...)`, no timeout/catchError) — unlike the already-hardened `chat_websocket_service.dart` |
| task-05 | WebView auth-challenge (`AttachmentPreviewView`) | `WebViewController()` at line 44 has **no `NavigationDelegate` at all** — confirmed correct via `rg`, see Correction Log above for how this was verified against a wrong hypothesis |
| task-06 | Android Play Core NPE | Cold-start NPE in `PlayCoreDialogWrapperActivity` — likely a transitive dependency of Flutter's deferred-components support, not a direct plugin; needs dependency-chain confirmation before fixing |
| task-07 | `Ticket.fromJson` null cast | `ticket.dart:54` casts `json['number'] as int` (and `id` on line 53) with no null-safety — backend occasionally returns null, failing the whole tickets-list parse inside a `compute()` isolate. Same family as the 4 crashes already fixed in `SE-11997` |
| task-08 | Crashlytics housekeeping | 3 issues already fixed in code (verified via git history), still shown OPEN — no code change, just close them |

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| Native iOS plugin (`PlatformUtil.swift`, vendored) | Unknown CocoaPod dependency | FATAL crash, confirmed regression in 1.7.1 field data |
| Ticket/chat/asset/notification screens (iOS) | Various Flutter/Dart code paths | Stack Overflow FATAL, repetitive per affected user |
| Push notification permission flow (iOS) | `NotificationManager.requestPermissions` | FATAL crash on permission-denied instead of graceful handling |
| `FirebirdSocketService` (Kabuto asset-live thumbnail socket) | Phoenix channel join/push | NON_FATAL exceptions on timeout/assertion, no guard |
| Attachment preview WebView (iOS) | `AttachmentPreviewView` | FATAL — auth challenge with zero delegate handling |
| Android cold start | Play Core / Flutter deferred-components (transitive) | FATAL NPE on some devices |
| Ticket list (Android + iOS) | Parses `GET /tickets` via `Ticket.fromJson` | Silent tickets-list load failure when `number`/`id` is null |
| Firebase Crashlytics console | Issue tracking | 3 stale OPEN issues creating false-positive noise for future triage |

## Scope

### In Scope
Eight tasks, one per root-cause cluster (see Task Summary). Regression tests where automatable (task-03, task-04, task-05, task-07); manual device/simulator verification substituting for native-platform crashes with no unit-test path (task-01, task-06); no code change for task-08 (Crashlytics API only).

### Out of Scope
- Any Crashlytics issue outside these 9 issue IDs / the current 90-day window. New issues surfacing later get their own task via `add-defect` or a follow-up plan — do not silently fold them in.
- `chat_websocket_service.dart` — already has guarded join/timeout/error-handling logic (verified during the original planning); not touched by task-04 (that task targets the separate, unguarded `freebird_websocket_service.dart`).
- `login_web_view.dart` — already fixed by `SE-12758` (2026-06-17); not touched by task-05 (that task targets the separate, unguarded `attachment_preview_view.dart`). See Correction Log.
- `android-r8-minification` plan's own ProGuard work — task-06 must coordinate on `proguard-rules.pro` (same file) but does not duplicate or resolve that plan's own task-03 (blocked, manual Play Store validation).
- **Genuinely not yet planned by anyone** (real gaps found during the 2026-09-07 audit, not part of this plan):
  - `FragmentStateManager.createView` FATAL (Android, top open Android issue by volume, 29 events/16 users) — root cause likely in Flutter's `FlutterActivity`/`FlutterFragment` embedding after the 3.44 SDK upgrade (`SE-12530`); needs its own embedding-focused investigation.
  - `ChatWebSocketService.startNewChat` "Failed to join channel" (Android `6cca598ad0eed80d3c1a344cb1d3af95`, iOS `f92cc8bc8266594c291fa984b0c75366`) — distinct file from `FirebirdSocketService` (task-04) and from the already-fixed `.connect` null-check; channel-join lifecycle specifically in the chat service. Not investigated by anyone yet.
  - Low-volume native/OS crashes not covered above (missing `libflutter.so`, ANRs, `SuperNotCalledException` from `flutter_inappwebview`, `RemoteServiceException`) — third-party/OS-level, low ROI, monitor only.

## Kill Criteria

- Fix introduces worse behavior than the original bug (e.g. silently dropping tickets instead of showing them with a placeholder)
- Root cause is disproved by new evidence once implementation starts (applies to any task, but especially task-02 and task-06 — see Risks)
- **task-01**: if `PlatformUtil.swift` traces back to a third-party CocoaPod with no released fix or version bump available, stop and escalate — do not patch vendored Pod source directly (unmaintainable, breaks on next `pod install`)
- **task-06**: if removing/pinning Play Core would disable a Flutter deferred-components / dynamic-feature capability the app actually uses (unconfirmed at planning time), confirm with the team before changing `build.gradle`. Coordinate with `android-r8-minification` before editing `proguard-rules.pro`
- **task-04**: if fixing `FirebirdSocketService` reveals it needs the same reconnect/backoff architecture as `chat_websocket_service.dart`, treat that as a scope increase — stop and re-scope as a separate technical plan rather than growing this bug-fix task into an infra rewrite

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|--------------|
| 1 | Confirmed Regression | task-01 | None | `ddf4c678` — already reproduced in 1.7.1, highest priority |
| 2 | Remaining Issues | task-02, task-03, task-04, task-05, task-06, task-07, task-08 | None | Independent fixes, no shared files — fully parallelizable among themselves and with task-01 |

Phase numbering reflects **priority**, not a technical dependency — Phase 2 tasks don't wait on Phase 1. Within Phase 2, task-07 (`Ticket.fromJson`) is prioritized just behind task-02 (top-volume iOS FATAL) because it's mechanical, low-risk, and matches an already-established fix pattern; task-08 (housekeeping) and task-06 (low-volume native) are lowest priority.

## Task Summary

| Task Path | Title | Phase | Status | Priority | Depends On |
|-----------|-------|-------|--------|----------|------------|
| phase-1/task-01-ios-platformutil-breakpoint | iOS: fix `PlatformUtil.init(plugin:)` EXC_BREAKPOINT regression | 1 | **complete (escalated)** | **HIGHEST** — confirmed regression, FATAL, reproduced in 1.7.1 field data | — |
| phase-2/task-02-ios-stack-overflow-investigation | iOS: investigate + fix repetitive FlutterError Stack Overflow | 2 | **blocked** (no symbolication; dormant since 1.7.0) | HIGH — top-volume open iOS issue (35 events/10 users), SIGNAL_REPETITIVE | — |
| phase-2/task-07-ticket-fromjson-nullsafety | Fix `Ticket.fromJson` null cast on `id`/`number` | 2 | not-started | HIGH — same pattern as 4 prior confirmed fixes, low risk | — |
| phase-2/task-03-notification-permission-crash | iOS: guard `NotificationManager.requestPermissions` against uncaught `firebase_messaging/unknown` | 2 | **complete** (pushed to bla, PR pending) | MEDIUM — SIGNAL_FRESH on the most recent instance | — |
| phase-2/task-04-freebird-socket-channel-errors | Guard `FirebirdSocketService` channel join/push against phoenix_socket timeout/assertion errors | 2 | not-started | MEDIUM | — |
| phase-2/task-05-webview-attachment-auth-challenge | Add `onHttpAuthRequest` to `AttachmentPreviewView`'s WebView (SE-12758 pattern, new call site) | 2 | not-started | MEDIUM — FATAL but low volume (2 events/2 users) | — |
| phase-2/task-06-android-play-core-npe | Android: fix `PlayCoreDialogWrapperActivity` NPE on cold start | 2 | not-started | LOW — low volume, likely third-party/transitive | — |
| phase-2/task-08-crashlytics-housekeeping | Close 3 stale Crashlytics issues already fixed in code | 2 | not-started | LOW — administrative, no code change | — |

## Branch Convention

Per-task branches: `plan/fix-production-crashes-v180/{phase}/{task-path}` (e.g. `plan/fix-production-crashes-v180/phase-1/task-01-ios-platformutil-breakpoint`), branched from `develop`.
Merge target: `develop`.
No shared branch — all 8 tasks are independent and touch disjoint files.

## Key Files

| File/Directory | Relevance |
|-----------------|-----------|
| `syncro-flutter/ios/Podfile.lock` | task-01 — identify which CocoaPod ships `PlatformUtil.swift` (not in `ios/Runner/`, confirmed plugin-vendored) |
| `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` | task-03 — `requestPermissions()` (line ~230), no try/catch |
| `syncro-flutter/lib/core/services/freebird_websocket_service.dart` | task-04 — `init()` (line ~63), no timeout/catchError on channel join |
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | Reference only (already hardened) — out of scope, do not modify |
| `syncro-flutter/lib/features/ticket/ticket_attachment/presentation/attachment_preview_view.dart` | task-05 — `WebViewController()` (line 44) has no `NavigationDelegate` at all |
| `syncro-flutter/lib/features/authentication/login/presentation/login_web_view.dart` | Reference only — SE-12758's fix (`onHttpAuthRequest`, line 114) is the pattern task-05 mirrors. Out of scope, do not modify |
| `syncro-flutter/android/app/build.gradle` | task-06 — `targetSdkVersion` (line 80); Play Core dependency investigation |
| `syncro-flutter/android/app/proguard-rules.pro` | task-06 — coordinate with `android-r8-minification` plan before editing |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart` | task-07 — lines 52-54 |
| `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_deserializer.dart` | task-07 context — where the isolate-level failure surfaces (line 10) |
| `syncro-flutter/test/features/ticket/ticket_home/domain/ticket_test.dart` | task-07 regression test |
| `.plans/completed/SE-12758-webview-auth-challenge/overview.md` | Precedent for task-05 — identical crash class, different call site |
| `docs/kb-projects/syncro-flutter/technical/integrations/firebase.md` | Relevant KB — task-01, task-02, task-03 |
| `docs/kb-projects/syncro-flutter/technical/integrations/phoenix-socket.md` | Relevant KB — task-04 |
| `docs/kb-projects/syncro-flutter/product/features/push-notifications.md` | Relevant KB — task-03 |

## Crashlytics Issue IDs

| Issue | Platform | ID | Type | Task |
|-------|----------|----|----|------|
| `PlatformUtil.init(plugin:)` EXC_BREAKPOINT | iOS | `ddf4c6780aa1a1d4452806dd8f5e381a` | **FATAL** (regressed) | task-01 |
| Stack Overflow (FlutterError) | iOS | `0bae269c5d2442a15a183ad4642da5c7` | **FATAL** | task-02 |
| `NotificationManager.requestPermissions` | iOS | `307055b68251cef77c20c6ca27b33816` | **FATAL** | task-03 |
| `NotificationManager.requestPermissions` | iOS | `4d17e7a500b300c48a84f037b2ded77a` | **FATAL** | task-03 |
| `FirebirdSocketService` phoenix_socket | iOS | `aa0323e5c16c3e7546f84452e28eb8a3` | NON_FATAL | task-04 |
| `FirebirdSocketService` phoenix_socket (`PhoenixChannel.join`) | iOS | `be06f8e7c72127f022185820c3d9ad4b` | NON_FATAL | task-04 |
| WebView auth-challenge (`AttachmentPreviewView`) | iOS | `b0914a639eb5e4b996dec3e8f1147a4b` | **FATAL** | task-05 |
| Play Core `PlayCoreDialogWrapperActivity` NPE | Android | `5a34e9320c9c52a9d8dc5d1e21b24d49` | **FATAL** | task-06 |
| `Ticket.fromJson` null cast | Android | `f91fb1b40d29c211bdf65655b75937b8` | NON_FATAL | task-07 |
| `Ticket.fromJson` null cast | iOS | `50306dfd3ee04e4d8c71ae4d212954dc` | NON_FATAL | task-07 |
| `asset_filter_deserializer` (stale, fixed in v152) | Android | `9a1d349f325abb563d2b26653a1b993c` | NON_FATAL | task-08 |
| `WorksheetTemplateDeserializer` (stale, fixed in v152) | Android | `de94f83f3e5808649141d2c1d40c58a8` | NON_FATAL | task-08 |
| `DioMixin` 504 (stale, backend-owned, out of scope) | Android | `7ec1fff22d998a861ff0d1705d05518d` | NON_FATAL | task-08 |

## Risks

- **task-01 and task-06 are native-platform crashes** (Swift/Kotlin-adjacent, not Dart) — neither has a unit-test path the way the other tasks do. Both substitute manual device/simulator verification for automated regression coverage; see each task's Testing section.
- **task-02's root cause is unknown at planning time.** The Crashlytics issue title (". - ...") carries no useful symbol info, and it spans firstSeen 1.4.0 → lastSeen 1.7.0. The task starts with a mandatory investigation step (pull the full stack trace via `crashlytics_batch_get_events`) before any fix is attempted — do not skip straight to a guess-and-check fix.
- **task-06's fix approach is a hypothesis, not yet confirmed.** No direct Play Core plugin dependency was found in `pubspec.yaml`; it's very likely a transitive dependency of the Flutter engine's deferred-components support. Confirm the actual dependency chain before picking a fix.
- **This plan's own scope-correction history** (see Correction Log) is a live example of a plausible-sounding but wrong root-cause hypothesis reaching HIGH confidence — task-02 in particular should stay disciplined about investigating before concluding.

## Success Criteria

- [ ] All 8 tasks complete
- [ ] Each Dart-level fix (task-03, task-04, task-05, task-07) has a passing regression test proving the previously-uncaught exception no longer propagates
- [ ] task-01 and task-06 verified manually on-device/simulator (documented in their `status.md`)
- [ ] `fvm flutter analyze` and `fvm flutter test` pass across the whole plan
- [ ] `pre-commit-check` passes on every task's commits
- [ ] No regression in `chat_websocket_service.dart` or `login_web_view.dart` (both explicitly out of scope, unmodified)
- [ ] The 3 stale Crashlytics issues (task-08) marked CLOSED
- [ ] 0 new occurrences of any of the 9 fixed issues in Crashlytics after release

## Defects

<!-- Bugs discovered during plan execution. Added by execute-task's Bug Discovery Protocol (Step 5a). -->

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

- [ ] All 8 tasks complete or adapted
- [ ] Bugs no longer reproducible with original repro steps
- [ ] Regression tests: red before fix, green after (verified)
- [ ] All existing tests pass
- [ ] Each task's root cause matches the actual fix (no drift)
- [ ] KB/documentation updated or explicitly marked not needed
- [ ] Staging verification complete

## Revert Plan

**Revert trigger**: New Crashlytics events for any fixed issue within 48h of release, or a new regression in the affected flow
**Revert steps**: Revert the merge commit for the affected task's branch; re-open the corresponding Crashlytics issue(s)
**Rollback owner**: Alex Salazar

## References

- **Ticket**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: [fix-production-crashes-v140](../fix-production-crashes-v140/overview.md) (SE-11997, same null-safety pattern as task-07), [fix-production-crashes-v152](../completed/fix-production-crashes-v152/overview.md) (asset_filter/worksheet issues being closed in task-08), [SE-12758-webview-auth-challenge](../completed/SE-12758-webview-auth-challenge/overview.md) (precedent for task-05), [android-r8-minification](../android-r8-minification/overview.md) (task-06 must coordinate on `proguard-rules.pro`)
