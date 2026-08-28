# Task: Android — fix `PlayCoreDialogWrapperActivity` NPE on cold start

**Plan**: 1.7.1 Crashlytics Fixes (1.7.1-crashlytics)
**Phase**: 2
**Task ID**: task-06
**Task Path**: `phase-2/task-06-android-play-core-npe`
**Depends On**: None
**JIRA**: N/A — create one if desired
**Crashlytics Issue**: `5a34e9320c9c52a9d8dc5d1e21b24d49` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:android:3352304fd0baa59e5b5c5b/issues/5a34e9320c9c52a9d8dc5d1e21b24d49)) — FATAL, SIGNAL_EARLY (100% of crashes happen in the first second of a session), firstSeen 1.5.0, lastSeen 1.7.1

## Objective

Fix `java.lang.NullPointerException: Attempt to invoke virtual method 'java.lang.Object android.os.Bundle.get(java.lang.String)' on a null object reference` in `com.google.android.play.core.common.PlayCoreDialogWrapperActivity.onCreate`. **This is a well-known upstream Flutter/Android issue** (Play Core's split-install dialog activity reading a null `Intent` extra on Android 12+, typically after the OS recreates the activity post process-death) — not something specific to this app's own code, but this app is still exposed to it.

## Context

**Hypothesis, not yet confirmed** (per plan overview's Risks section): `pubspec.yaml` has **no direct dependency** on `in_app_update`, `play_core`, or any plugin whose name suggests it (confirmed via grep during planning). Play Core is very likely pulled in **transitively** — either by the Flutter engine itself (deferred-components / Play Feature Delivery support, bundled by default even when unused) or by another plugin (Firebase, Google Sign-In-adjacent, etc.).

This task starts with confirming the actual dependency chain before picking a fix — do not guess at a `build.gradle` change without first knowing which dependency brings Play Core in.

Also relevant: `android/app/build.gradle` line ~80 currently reads `targetSdkVersion flutter.targetSdkVersion` (delegates to the Flutter tool's default, not pinned in-app) — this crash is specifically an Android 12+ (API 31+) issue, so confirm the effective `targetSdkVersion` as part of the investigation.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase bla develop`
- [ ] Create the task branch: `git switch -c plan/1.7.1-crashlytics/phase-2/task-06-android-play-core-npe`
- [ ] **Check `.plans/android-r8-minification/overview.md` and its task-03/task-04 status** before touching `proguard-rules.pro` — that plan is in-progress (3/4, task-03 blocked on manual Play Store validation) and shares this file. Coordinate, don't clobber.
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/android/app/build.gradle` | modify (conditional) | Depends on Step 1's findings — may need an explicit Play Core version pin/exclusion |
| `syncro-flutter/android/app/proguard-rules.pro` | modify (conditional) | Only if a ProGuard/R8 keep-rule is the chosen fix — coordinate with `android-r8-minification` plan first (see Before You Start) |

### Do NOT Modify

- Anything under `syncro-flutter/lib/`, `syncro-flutter/ios/` — Android-native issue only
- `android-r8-minification` plan's own task files — this task coordinates with that plan, it doesn't merge into it

## Implementation Steps

### Step 1: Confirm the dependency chain (mandatory before any fix)
Run the Android dependency tree (e.g. `./gradlew :app:dependencies` from `android/`, or inspect the resolved `com.google.android.play:core*` artifacts) to find exactly which dependency pulls in Play Core, and its version. Confirm effective `targetSdkVersion`.

### Step 2: Pick the fix based on Step 1
Typical known fixes for this exact crash (confirm which applies once the dependency chain is known — do not apply blind):
- Bump the transitive Play Core dependency to a version where this is fixed (Play Core ≥1.10.3 addressed related null-extra handling).
- If the app has zero use of Play Feature Delivery / deferred components (likely, given no direct plugin dependency), consider excluding the transitive Play Core split-install module rather than patching around it — confirm with the team first per the plan's kill criteria (don't remove a capability without confirming it's actually unused).
- A ProGuard/R8 keep-rule workaround, if that's what upstream guidance recommends for the confirmed version — coordinate with `android-r8-minification`.

### Step 3: Verify
Manual verification only (see Testing) — this is a framework/OS-level NPE in vendored Play Core code, not app logic, so it has no unit-test path.

## Testing

- [ ] No automated regression test is feasible for this crash (native Android framework code, not app code) — known coverage gap, called out in the plan overview's Risks section.
- [ ] Manual verification: cold-start the app on an Android 12+ device/emulator (matching the SIGNAL_EARLY "first second of session" pattern) multiple times, confirm no crash. Cross-reference with `android-r8-minification`'s existing manual Play Store validation step (task-03) if the fix touches shared release-build configuration.
- [ ] `./gradlew assembleRelease` (or the project's equivalent release-mode build command) succeeds
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Add a short note to `docs/kb-projects/syncro-flutter/engineering/best-practices/flutter-standards.md` or a new Android-integrations doc, documenting the confirmed Play Core dependency source and fix — this is exactly the kind of non-obvious, easy-to-rediscover-the-hard-way finding `document-solution` exists for. Run `check-kb-index` after.

## Completion Criteria

- [ ] Play Core dependency chain confirmed and documented in `status.md`
- [ ] Fix applied per Step 2, coordinated with `android-r8-minification` if `proguard-rules.pro` was touched
- [ ] Manual cold-start verification performed and recorded
- [ ] No new occurrence of this issue in Crashlytics after the fix ships (follow-up check, not blocking this task's completion)
