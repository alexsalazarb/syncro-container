# Status: iOS — fix `PlatformUtil.init(plugin:)` EXC_BREAKPOINT regression

**Current Status**: complete (escalated — closed per Kill Criteria, no vendored-Pod patch applied)
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: `plan/fix-production-crashes-v180/phase-1/task-01-ios-platformutil-breakpoint` (syncro-flutter submodule, based on `develop`)
**PR**: N/A (PR_INTEGRATION=false)

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | complete | claude | Investigated, confirmed third-party bug with no released fix, escalated per Kill Criteria — see findings below |

## Findings

**Step 1 (identify the plugin)**: `PlatformUtil.swift` belongs to `flutter_inappwebview_ios` **1.1.2** (verified locally at `~/.pub-cache/hosted/pub.dev/flutter_inappwebview_ios-1.1.2/ios/Classes/PlatformUtil.swift`), a **transitive** dependency pulled in by `oauth_webauth: ^5.1.0` (resolved `5.2.0` in `pubspec.lock`), which this app uses in `login_web_view.dart`'s `handleOAuth()` → `BaseWebScreen.start(...)`. None of the plugins guessed in the task's Context section (`mobile_scanner`, `local_auth`, `permission_handler`, etc.) were the actual source.

**Root cause confirmed**: `PlatformUtil.swift:15`:
```swift
init(plugin: SwiftFlutterPlugin) {
    super.init(channel: FlutterMethodChannel(name: PlatformUtil.METHOD_CHANNEL_NAME, binaryMessenger: plugin.registrar!.messenger()))
    ...
}
```
Force-unwraps `plugin.registrar!` — if `registrar` is `nil` at this point, Swift traps with `EXC_BREAKPOINT`. Our Crashlytics stack trace (`b3d28bebfd68471f9629e875689f4b42_2256804945915234800`, iOS 26.6.0, v1.7.1 build 443) confirms the exact call chain: `AppDelegate.application(didFinishLaunchingWithOptions:)` → `GeneratedPluginRegistrant.registerWithRegistry:` → `SwiftFlutterPlugin.register(with:)` → `SwiftFlutterPlugin.init(with:)` → `PlatformUtil.init(plugin:)` — i.e. this crashes during **initial plugin registration at cold start**, not during later use.

**This is a confirmed, currently-unfixed upstream bug**, not something introduced by our code:
- GitHub issue [`pichillilorenzo/flutter_inappwebview#2368`](https://github.com/pichillilorenzo/flutter_inappwebview/issues/2368) ("[Crash][iOS] Swift runtime failure: Unexpectedly found nil while unwrapping an Optional value") reports the **exact same crash** at `PlatformUtil.swift:15` during `SwiftFlutterPlugin.init(with:)`. Opened 2024-10-24, still open as of this investigation, **no merged fix or linked PR**.
- `flutter_inappwebview` **6.1.5** (our resolved version) is still the latest **stable** release on pub.dev (confirmed via package page — no newer stable exists). A `6.2.0-beta.3` prerelease exists but is beta-only; `oauth_webauth`'s constraint (`^6.0.0`) would technically allow it, but pinning production to a beta dependency is not an acceptable substitute for a real fix.
- Per the plan's Kill Criteria: *"if `PlatformUtil.swift` traces back to a third-party CocoaPod with no released fix or version bump available, stop and escalate — do not patch vendored Pod source directly (unmaintainable, breaks on next `pod install`)."* Both conditions are met. No code change made.

**Unconfirmed lead worth flagging, not verified**: the crash title shows `[Runner.debug.dylib]` and the upstream GitHub issue reporter noted this reproduces with `flutter build ipa --debug` but not `--release`. I could not confirm whether our production release pipeline is somehow producing a debug-configuration binary — `syncro-create-release`'s actual build script isn't present in this checkout to inspect (only a `WINDSURF.md` stub, no real skill content), and I did not attempt an actual release build to test this (too invasive/slow for this investigation pass). **This is worth a human checking directly** — if our release pipeline is inadvertently archiving with `-configuration Debug` (or an Xcode scheme misconfiguration), that would be a real, our-side fix, independent of the upstream bug. Also possible `Runner.debug.dylib` is just how Crashlytics labels a Swift debug-info dylib bundled even in genuine Release archives — not confirmed either way.

## Actions taken

- Added a note to Crashlytics issue `ddf4c6780aa1a1d4452806dd8f5e381a` documenting this finding (see Artifacts)
- Added a comment to JIRA `SE-13805` documenting the escalation (see Artifacts)
- Did NOT comment on the upstream GitHub issue #2368 — posting on behalf of the team wasn't authorized in this session; flagged as a suggested next step for a human to do if useful

## Blockers

None — this is a closed/escalated task, not a stalled one. Follow-up ownership: someone should (a) verify the release-build-configuration lead above, and (b) periodically check `pichillilorenzo/flutter_inappwebview` for a stable release fixing issue #2368, then revisit.

## Artifacts

- Crashlytics note on `ddf4c6780aa1a1d4452806dd8f5e381a`
- JIRA comment on [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)
- Local task branch `plan/fix-production-crashes-v180/phase-1/task-01-ios-platformutil-breakpoint` in `syncro-flutter` (no code changes — investigation only, not pushed pending user confirmation)

## Adaptations

**Adapted**: task.md's Context section guessed at candidate plugins (`mobile_scanner`, `local_auth`, `permission_handler_apple`, `flutter_local_notifications`, `device_info_plus`, `app_badge_plus`, `connectivity_plus`) — none were correct. Actual source is `flutter_inappwebview_ios` (transitive via `oauth_webauth`), not directly declared in `pubspec.yaml`. Also: task.md's Implementation Steps assumed a fix (version bump or Podfile pin) would be reachable; investigation found the Kill Criteria's escalation clause applies instead — no fix was implemented, per the plan's own "either is an acceptable close condition" completion rule.
