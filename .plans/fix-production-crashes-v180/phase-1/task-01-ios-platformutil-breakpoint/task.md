# Task: iOS — fix `PlatformUtil.init(plugin:)` EXC_BREAKPOINT regression

**Plan**: Fix Production Crashes — v1.8.0 (fix-production-crashes-v180)
**Phase**: 1
**Task ID**: task-01
**Task Path**: `phase-1/task-01-ios-platformutil-breakpoint`
**Depends On**: None
**JIRA**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
**Crashlytics Issue**: `ddf4c6780aa1a1d4452806dd8f5e381a` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/ddf4c6780aa1a1d4452806dd8f5e381a))

## Objective

Fix the `EXC_BREAKPOINT` crash in `[Runner.debug.dylib] PlatformUtil.swift - PlatformUtil.init(plugin:)`. This is the **highest-priority task in the plan** — it was previously closed (2026-06-16), regressed in 1.7.0 (build 439, 2026-06-25), and has a **confirmed event in 1.7.1** (`lastSeenVersion: 1.7.1`), the build published 2026-08-27.

## Context

`PlatformUtil.swift` does **not** exist in this app's own iOS source (`syncro-flutter/ios/Runner/` contains only `AppDelegate.swift` — verified during planning). It is vendored plugin code, pulled in via CocoaPods. `EXC_BREAKPOINT` in Swift is almost always a `fatalError()`, a failed `try!`, or a force-unwrapped `nil` Optional — i.e. the plugin's own code hitting a precondition it assumes always holds.

This task starts with identifying the plugin before attempting any fix:

1. Search `ios/Podfile.lock` for pods with a `PlatformUtil`-style helper. Likely candidates given this app's `pubspec.yaml` dependencies: `mobile_scanner`, `local_auth`, `permission_handler`, `flutter_local_notifications`, `device_info_plus`, `app_badge_plus`, `connectivity_plus` (unverified — confirm via Podfile.lock, don't guess).
2. Once identified, check the plugin's GitHub issues/changelog for a known `EXC_BREAKPOINT` / `PlatformUtil.init(plugin:)` report matching the symbol.
3. Pull the full stack trace via `mcp__firebase__crashlytics_batch_get_events` on the sample event for this issue (`b3d28bebfd68471f9629e875689f4b42_2256804945915234800`, from the 2026-08-28 report run) — re-confirm it's still the current sample via `mcp__firebase__crashlytics_get_issue(appId, issueId: "ddf4c6780aa1a1d4452806dd8f5e381a")` first, since a new sample may have superseded it — for the exact call site and any custom keys/logs attached.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase bla develop`
- [ ] Create the task branch: `git switch -c plan/fix-production-crashes-v180/phase-1/task-01-ios-platformutil-breakpoint`
- [ ] Confirm the sample event id and pull the full stack trace (see Context step 3) before touching any code
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/ios/Podfile.lock` | read | Identify the offending pod |
| `syncro-flutter/pubspec.yaml` | modify (conditional) | Only if the fix is a version bump of the identified plugin |
| `syncro-flutter/ios/Podfile` | modify (conditional) | Only if a pin/workaround is needed |

### Do NOT Modify

- Anything under `syncro-flutter/lib/` — this is a native-only crash, no Dart call site is implicated unless the investigation proves otherwise (if it does, update this task's File Ownership and note the deviation in `status.md`)
- `syncro-flutter/ios/Runner/AppDelegate.swift` — not implicated unless investigation says otherwise

## Implementation Steps

### Step 1: Identify the plugin
Per Context above. Do not proceed to a fix until the specific plugin + version is confirmed.

### Step 2: Determine fix path
- If a newer plugin version fixes it upstream → bump in `pubspec.yaml`, run `fvm flutter pub get`, regenerate `ios/Podfile.lock` (`pod install` in `ios/`).
- If no upstream fix exists → apply the kill criterion in the plan overview: escalate, do not patch vendored Pod source.

### Step 3: Verify
Manual verification only (see Testing) — this crash cannot be reliably unit-tested since it originates inside vendored native plugin code.

## Testing

- [ ] No automated regression test is feasible for this native crash (vendored plugin internals, not app code) — this is a known coverage gap, called out in the plan overview's Risks section, not an oversight.
- [ ] Manual verification: exercise whatever app feature invokes the identified plugin, on a physical device or simulator running iOS 26, both before (confirm repro if possible) and after the fix.
- [ ] `fvm flutter analyze` passes (in case `pubspec.yaml`/Dart-adjacent changes were needed)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] If the root cause turns out to be a well-understood plugin issue, add a short note to `docs/kb-projects/syncro-flutter/technical/integrations/firebase.md` or create a new integration doc for the implicated plugin, capturing the crash signature for future Crashlytics triage. Run `check-kb-index` after.

## Completion Criteria

- [ ] Offending plugin identified and documented in `status.md`
- [ ] Fix applied (version bump or documented escalation per kill criteria — either is an acceptable close condition)
- [ ] Manual verification performed and recorded
- [ ] No new `EXC_BREAKPOINT` in `PlatformUtil.init(plugin:)` observed in the next release's Crashlytics data (follow-up check, not blocking this task's completion)
