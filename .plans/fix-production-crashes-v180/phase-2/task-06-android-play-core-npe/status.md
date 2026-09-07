# Status: Android — fix `PlayCoreDialogWrapperActivity` NPE on cold start

**Current Status**: blocked
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: — (no code branch — investigation only, no fix applied pending product confirmation)
**PR**: N/A (PR_INTEGRATION=false)

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | blocked | claude | Dependency chain confirmed, candidate fix identified, but withheld pending product confirmation on Pendo's in-app review usage — see Findings |

## Findings

**Step 1 (confirm dependency chain)** — done via `./gradlew :app:dependencies --configuration productionReleaseRuntimeClasspath` (had to discover the flavor-specific config name first: this app has `production`/`qa` flavors, not the plain `release` I initially assumed):

```
project :pendo_sdk
  +--- com.google.android.play:review:2.0.1
  |     \--- com.google.android.play:core-common:2.0.2
  +--- com.google.android.play:review-ktx:2.0.1
```

**`PlayCoreDialogWrapperActivity` comes from Pendo SDK**, transitively via its Play In-App Review integration (`com.google.android.play:review`) — not from any direct dependency of this app (confirmed no direct `play-core`/`in_app_review`/`in_app_update` plugin in `pubspec.yaml`, matching the task's hypothesis). Confirmed the activity is genuinely declared in `core-common:2.0.2`'s own `AndroidManifest.xml` (extracted the cached AAR and grepped it directly) — it merges into our app's manifest via Gradle's manifest merger, whether or not we ever invoke it.

**Effective `targetSdkVersion`**: 35 (confirmed via the Flutter Gradle plugin's `flutter.targetSdkVersion` default for Flutter 3.44.1) — well above API 31, so this app is fully exposed to the Android 12+-specific version of this bug.

**Version bump is not a viable fix**: researched via GitHub — `britannio/in_app_review#124` and `#139` both report this exact crash ("PlayCoreDialogWrapperActivity crash") specifically **against `core-common` 2.0.9**, and the fix that closed both issues (PR #143) was to *downgrade* to **2.0.2** — the exact version this app already resolves to. There is no newer version to bump to that's known to fix this; if anything, newer minor versions have their own reports of the same crash.

**Candidate fix identified, not applied**: the standard Android workaround for an unwanted auto-merged manifest component is a `tools:node="remove"` override in the app's own `AndroidManifest.xml`, targeting `com.google.android.play.core.common.PlayCoreDialogWrapperActivity` — this prevents the activity from ever being registered in the final merged manifest, so the OS/Play Store can't instantiate it (no component to crash). Low-risk, surgical, doesn't touch `build.gradle` or `proguard-rules.pro` (irrelevant anyway since `minifyEnabled false` per `android-r8-minification` plan).

**Why not applied**: `rg` across `lib/` and `android/app/src/` found **zero** code references to any in-app-review trigger (`requestReview`, `ReviewManager`, `launchReviewFlow`, etc.) — but Pendo's in-app review feature can be triggered remotely via Pendo's own dashboard/guide configuration, invisible to a code search. Removing the activity would silently break that capability if Pendo (or the team) is relying on it. Per the plan's own Kill Criteria ("confirm with the team before changing build.gradle" — extending the same caution to this manifest-level removal, since the risk shape is identical: removing a capability without confirming it's unused) and per direct confirmation from Alex ("no estoy seguro"), **withholding the fix** rather than guessing.

## Recommendation

Before applying the `tools:node="remove"` manifest override:
1. Check Pendo's dashboard/guide configuration for any active in-app-review campaign, or ask whoever manages the Pendo account.
2. If confirmed unused, this becomes a low-risk one-line manifest change — re-open this task.
3. If it IS used, this task should escalate as unfixable-without-product-tradeoff, same disposition as task-01.

## Blockers

Waiting on product/Pendo-configuration confirmation (see Recommendation) before a fix can be safely applied.

## Artifacts

None — no code changes made.

## Adaptations

None — task.md's own hypothesis and instructions were followed exactly (confirm dependency chain first, don't guess a fix, confirm with the team before removing a capability). The investigation reached a concrete, ready-to-apply candidate fix; only the go/no-go decision is pending.
