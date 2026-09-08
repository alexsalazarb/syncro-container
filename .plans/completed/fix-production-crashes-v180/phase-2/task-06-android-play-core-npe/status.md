# Status: Android — fix `PlayCoreDialogWrapperActivity` NPE on cold start

**Current Status**: complete (adapted — see below)
**Last Updated**: 2026-09-08
**Agent**: claude
**Branch**: `plan/fix-production-crashes-v180` (syncro-flutter submodule, based on `develop`) — unified plan branch
**PR**: N/A (PR_INTEGRATION=false) — see plan overview for the current PR link

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | blocked | claude | Dependency chain confirmed (Pendo SDK), candidate fix identified, withheld pending confirmation Pendo's in-app review isn't in active use |
| 2026-09-08 | complete | claude | User confirmed (personal usage signal, not a dashboard check) never seeing the native review popup; accepted the residual, monitorable risk. Fix applied and verified |

## Findings (carried over from the blocked investigation)

See the original findings below for the full dependency-chain investigation (Pendo SDK → `com.google.android.play:review:2.0.1` → `core-common:2.0.2`, confirmed via `./gradlew :app:dependencies --configuration productionReleaseRuntimeClasspath`, confirmed effective `targetSdkVersion` 35, confirmed via GitHub research that no newer `core-common` version fixes this).

## Decision on the residual risk

Discussed directly with Alex before applying: `tools:node="remove"` guarantees the current NPE stops (the component won't exist to crash). It does NOT guarantee zero risk if Pendo's SDK ever actively calls Play Core's `launchReviewFlow()` — that would throw `ActivityNotFoundException` instead, a different crash, only if Pendo doesn't already catch it (couldn't verify Pendo's obfuscated `.dex` without a decompiler — assessed as disproportionate effort for a LOW-priority task). Alex confirmed he's personally never seen the native "rate this app" popup (a suggestive but not dashboard-confirmed signal) and accepted the asymmetry: today's crash is guaranteed and recurring; the alternative risk is hypothetical, low-probability, and — critically — trivially detectable and revertible (a new `ActivityNotFoundException` Crashlytics issue would immediately point at Pendo's review flow, at which point this override should be reverted and a different fix found, e.g. a real in-app review integration this app owns, or asking Pendo support directly).

## Fix Applied

`android/app/src/main/AndroidManifest.xml`: added `xmlns:tools` namespace + an explicit
```xml
<activity android:name="com.google.android.play.core.common.PlayCoreDialogWrapperActivity"
    tools:node="remove" />
```

**Verified, not assumed**: ran `./gradlew :app:processProductionReleaseMainManifest` and inspected both the merged manifest output (`build/app/intermediates/merged_manifest/productionRelease/.../AndroidManifest.xml` — confirmed zero occurrences of the activity) and the manifest merger report (`build/app/outputs/logs/manifest-merger-production-release-report.txt` — shows the library's declaration explicitly `REJECTED from [com.google.android.play:core-common:2.0.2]`, our override `ADDED`). This is a real, confirmed removal, not a hopeful guess. Hit and fixed one authoring mistake along the way: XML comments cannot contain a literal `--`, which broke the initial manifest merge attempt — caught immediately via the Gradle error, not silently.

## Verification

- [x] Manifest merge confirms the component is removed (see above — this is this task's only available "test", per its own Testing section: no automated regression test is feasible for a framework/OS-level native crash)
- [ ] Full `./gradlew assembleRelease` (or equivalent) — not run; would require release signing setup not exercised in this session. Manifest-merge-level verification is the strongest check available without a real release build/device test.
- [ ] Manual cold-start verification on an Android 12+ device — not done in this session (no device/emulator available); recommend before/shortly after this ships to production
- [ ] Follow-up: monitor Crashlytics after release for (a) confirmation this issue (`5a34e9320c9c52a9d8dc5d1e21b24d49`) stops recurring, and (b) any new `ActivityNotFoundException` issue that would indicate Pendo's review flow was actually in use

## Artifacts

- `android/app/src/main/AndroidManifest.xml` — fix
- Committed on the unified `plan/fix-production-crashes-v180` branch, pushed to `bla`

## Adaptations

**Adapted**: task.md's Completion Criteria asked for "manual cold-start verification performed and recorded" — not done here (no Android device/emulator in this session). Substituted the manifest-merge-report verification (confirms the fix mechanism works at the build level) but flagging that on-device confirmation is still outstanding and should happen before/around this release. Also: the go/no-go on the Pendo risk was resolved via a direct conversation with Alex (personal-usage signal, explicitly accepting the monitorable residual risk) rather than a Pendo dashboard check or vendor confirmation — a lighter-weight resolution than task.md's Kill Criteria implied, documented here for traceability if this decision needs revisiting.
