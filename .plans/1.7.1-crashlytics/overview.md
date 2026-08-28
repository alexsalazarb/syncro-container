# Plan: 1.7.1 Crashlytics Fixes

**Status**: not-started
**Created**: 2026-08-28
**Last Updated**: 2026-08-28
**Estimated Demo Date**: TBD — none requested
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Base Branch**: develop — confirmed via `git branch --show-current` in `syncro-flutter` (matches existing plan convention, overrides container AGENTS.md's stated `main`)

## Objective

Fix the 8 open Crashlytics issues (7 iOS, 1 Android) identified in the live 7-day report (2026-08-21 → 2026-08-28) for syncro-flutter, Firebase project `syncromsp-ios`, covering both apps:
- iOS: `1:920223298498:ios:a0d84c75923c83b35b5c5b`
- Android: `1:920223298498:android:3352304fd0baa59e5b5c5b`

v1.7.1 was published 2026-08-27. One issue (`ddf4c6780aa1a1d4452806dd8f5e381a`) is a **confirmed regression already reproduced in 1.7.1** — this drives Phase 1's priority ordering. The rest are not yet confirmed in 1.7.1 (mostly last seen in 1.7.0 or earlier) but are real, open, and grouped here so the team doesn't have to re-derive root causes issue by issue.

## Scope

### In Scope

Six tasks, one per root cause cluster, covering all 8 Crashlytics issue IDs:

| Task | Crashlytics Issue(s) | Root cause area |
|------|----------------------|------------------|
| task-01 | `ddf4c6780aa1a1d4452806dd8f5e381a` | Native iOS plugin (`PlatformUtil.swift`), EXC_BREAKPOINT, **regressed in 1.7.1** |
| task-02 | `0bae269c5d2442a15a183ad4642da5c7` | FlutterError Stack Overflow, repetitive on 2 users — root cause unknown, needs stack-trace investigation |
| task-03 | `307055b68251cef77c20c6ca27b33816`, `4d17e7a500b300c48a84f037b2ded77a` | `NotificationManager.requestPermissions` — uncaught `firebase_messaging/unknown` exception |
| task-04 | `aa0323e5c16c3e7546f84452e28eb8a3`, `be06f8e7c72127f022185820c3d9ad4b` | `FirebirdSocketService` (Kabuto asset-live thumbnail socket) — unguarded `phoenix_socket` channel join/push |
| task-05 | `b0914a639eb5e4b996dec3e8f1147a4b` | `AttachmentPreviewView`'s `WebViewController` — missing `onHttpAuthRequest` (same class of bug as SE-12758, different call site) |
| task-06 | `5a34e9320c9c52a9d8dc5d1e21b24d49` | Android `PlayCoreDialogWrapperActivity` NPE on cold start — known Play Core / Flutter deferred-components issue |

Every task owns its regression-test coverage where automatable (task-03, task-04, task-05 are Dart-level and testable; task-01 and task-06 are native-platform crashes with no unit-test path — see each task's Testing section for the manual-verification substitute).

### Out of Scope

- Any Crashlytics issue outside this 7-day window / outside these 8 issue IDs. If new issues surface after 1.7.1 field feedback comes in, they get their own task via `add-defect` or a follow-up plan — do not silently fold them into this one.
- `chat_websocket_service.dart` — already has guarded join/timeout/error-handling logic (verified during planning); out of scope, not touched by task-04.
- `login_web_view.dart` — already fixed by SE-12758 (2026-06-17); out of scope, not touched by task-05.
- `android-r8-minification` plan's own ProGuard work — task-06 must coordinate with it (same `proguard-rules.pro` file) but does not duplicate or resolve that plan's task-03 (blocked, manual Play Store validation).

## Kill Criteria

- **task-01**: if `PlatformUtil.swift` traces back to a third-party CocoaPod with no released fix or version bump available, stop and escalate — do not patch vendored Pod source directly (unmaintainable, breaks on next `pod install`).
- **task-06**: if removing/pinning Play Core would disable a Flutter deferred-components / dynamic-feature capability the app is actually using (unconfirmed at planning time — `pubspec.yaml` shows no direct plugin pulling it in), confirm with the team before changing `build.gradle`. Coordinate with `android-r8-minification` (in-progress, blocked on task-03) before editing `proguard-rules.pro` — don't let the two plans clobber each other's rules.
- **task-04**: if fixing `FirebirdSocketService` reveals it needs the same reconnect/backoff architecture as `chat_websocket_service.dart`, treat that as a scope increase — stop and re-scope as a separate technical plan rather than growing this bug-fix task into an infra rewrite.

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|--------------|
| 1 | Confirmed Regression | task-01 | None | `ddf4c678` — already reproduced in 1.7.1, highest priority |
| 2 | Remaining Issues | task-02, task-03, task-04, task-05, task-06 | None | Independent fixes, no shared files — fully parallelizable among themselves and with task-01 |

Phase numbering here reflects **priority**, not a technical dependency — Phase 2 tasks don't wait on Phase 1 to finish. task-01 is called out first because it's the only issue with confirmed 1.7.1 field evidence; everything else is prioritized by signal (SIGNAL_REGRESSED > SIGNAL_REPETITIVE > SIGNAL_FRESH/SIGNAL_EARLY > no signal) within Phase 2, not by phase number.

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| phase-1/task-01-ios-platformutil-breakpoint | iOS: fix `PlatformUtil.init(plugin:)` EXC_BREAKPOINT regression | 1 | not-started | — |
| phase-2/task-02-ios-stack-overflow-investigation | iOS: investigate + fix repetitive FlutterError Stack Overflow | 2 | not-started | — |
| phase-2/task-03-notification-permission-crash | iOS: guard `NotificationManager.requestPermissions` against uncaught `firebase_messaging/unknown` | 2 | not-started | — |
| phase-2/task-04-freebird-socket-channel-errors | Guard `FirebirdSocketService` channel join/push against phoenix_socket timeout/assertion errors | 2 | not-started | — |
| phase-2/task-05-webview-attachment-auth-challenge | Add `onHttpAuthRequest` to `AttachmentPreviewView`'s WebView (SE-12758 pattern, new call site) | 2 | not-started | — |
| phase-2/task-06-android-play-core-npe | Android: fix `PlayCoreDialogWrapperActivity` NPE on cold start | 2 | not-started | — |

## Branch Convention

Per-task branches: `plan/1.7.1-crashlytics/{phase}/{task-path}` (e.g. `plan/1.7.1-crashlytics/phase-1/task-01-ios-platformutil-breakpoint`), branched from `develop`. No shared branch — all 6 tasks are independent and touch disjoint files, so there's no reason to serialize them onto one branch.

## Key Files

| File/Directory | Relevance |
|-----------------|-----------|
| `syncro-flutter/ios/Podfile.lock` | task-01 — identify which CocoaPod ships `PlatformUtil.swift` (not in app's own `ios/Runner/` — confirmed only `AppDelegate.swift` exists there; it's plugin-vendored code) |
| `syncro-flutter/lib/core/services/push_notification/notifications_manager.dart` | task-03 — `requestPermissions()` (line ~230) calls `messaging.requestPermission(...)` with no try/catch |
| `syncro-flutter/lib/core/services/freebird_websocket_service.dart` | task-04 — `init()` (line ~63) calls `_channel.join().future.then(...)` with no timeout/catchError, unlike the hardened `chat_websocket_service.dart` |
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | Reference only (already hardened — lines 340-360 show the timeout+catch pattern task-04 should mirror). Out of scope, do not modify. |
| `syncro-flutter/lib/features/ticket/ticket_attachment/presentation/attachment_preview_view.dart` | task-05 — `WebViewController()` (line 44) has no `NavigationDelegate` at all |
| `syncro-flutter/lib/features/authentication/login/presentation/login_web_view.dart` | Reference only — SE-12758's fix (`onHttpAuthRequest`, line 114) is the pattern task-05 mirrors. Out of scope, do not modify. |
| `syncro-flutter/android/app/build.gradle` | task-06 — `targetSdkVersion flutter.targetSdkVersion` (line 80); Play Core version/dependency investigation |
| `syncro-flutter/android/app/proguard-rules.pro` | task-06 — coordinate with `android-r8-minification` plan before editing |
| `.plans/completed/SE-12758-webview-auth-challenge/overview.md` | Precedent for task-05 — identical crash class, different call site |
| `docs/kb-projects/syncro-flutter/technical/integrations/firebase.md` | Relevant KB — task-01, task-02, task-03 (Crashlytics, FCM) |
| `docs/kb-projects/syncro-flutter/technical/integrations/phoenix-socket.md` | Relevant KB — task-04 |
| `docs/kb-projects/syncro-flutter/product/features/push-notifications.md` | Relevant KB — task-03 |

## Risks

- **task-01 and task-06 are native-platform crashes** (Swift/Kotlin-adjacent, not Dart) — neither has a unit-test path the way the other four tasks do. Both substitute manual device/simulator verification for automated regression coverage; see each task's Testing section. This is a real coverage gap the team should know about, not swept under the rug.
- **task-02's root cause is unknown at planning time.** The Crashlytics issue title (". - ...") carries no useful symbol info, and it spans firstSeen 1.4.0 → lastSeen 1.7.0 (i.e., survived multiple releases). The task starts with a mandatory investigation step (pull the full stack trace via `crashlytics_batch_get_events` on the sample event) before any fix is attempted — do not skip straight to a guess-and-check fix.
- **task-06's fix approach is a hypothesis, not yet confirmed.** No direct Play Core plugin dependency was found in `pubspec.yaml`; it's very likely a transitive dependency of the Flutter engine's deferred-components support (a well-documented upstream Flutter/Android issue on Android 12+). The task starts with confirming the actual dependency chain (`./gradlew :app:dependencies` or equivalent) before picking a fix.

## Success Criteria

- [ ] All 6 tasks complete
- [ ] Each Dart-level fix (task-03, task-04, task-05) has a passing regression test proving the previously-uncaught exception no longer propagates
- [ ] task-01 and task-06 verified manually on-device/simulator (documented in their `status.md`)
- [ ] `fvm flutter analyze` and `fvm flutter test` pass across the whole plan
- [ ] `pre-commit-check` passes on every task's commits
- [ ] No regression in `chat_websocket_service.dart` or `login_web_view.dart` (both explicitly out of scope, unmodified)
