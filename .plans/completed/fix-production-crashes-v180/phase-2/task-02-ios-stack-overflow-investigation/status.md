# Status: iOS — investigate + fix repetitive FlutterError Stack Overflow

**Current Status**: blocked
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: — (no code branch created — investigation only, no fix to commit)
**PR**: N/A (PR_INTEGRATION=false)

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | blocked | claude | Investigated per Step 1 — root cause not identified, no symbolication available. See investigation.md |

## Blockers

**Cannot identify root cause without Dart symbolication.** The crash stack is raw `_kDartIsolateSnapshotInstructions+0x...` offsets with no function names, and no `--split-debug-info`/`--obfuscate` build artifact exists for the affected builds (1.4.0-1.7.0) in this checkout or any tracked CI config. Ruled out the two files a breadcrumb correlation pointed to (`chat_websocket_service.dart`, `attachment_preview_view.dart` + related ticket_attachment screens) — no recursive call path found in either. The breadcrumb correlation itself is weak: the `new_open_attachments_detail` event name isn't fired anywhere in our own code (likely Pendo-synthesized), so it may not map cleanly to the files inspected.

**However**: `crashlytics_get_report(topVersions)` shows **0 events on 1.7.1 and 1.8.0** (the two most recent releases) — all 35 events in the 90-day window are on 1.7.0 and 1.6.1. This has not recurred in ~2.5 months. Given that plus the symbolication blocker, further investigation right now has poor ROI — see investigation.md Recommendation.

**Unblock path**: either (a) someone locates an archived `--split-debug-info` symbol map for build 441/433 to decode the existing stack trace, or (b) the team adds that flag to the release pipeline going forward so a *future* recurrence (if any) is symbolicate-able, at which point this becomes tractable.

## Artifacts

- [investigation.md](investigation.md) — full findings, ruled-out files, version-recurrence data
- JIRA comment on SE-13805 documenting this status

## Adaptations

**Adapted**: Could not complete Step 2 (Fix) or satisfy the task's stated Completion Criteria ("investigation.md written with confirmed root cause" + "fix applied") — root cause was not confirmed despite following Step 1 in full (issue re-check, full stack trace pull, topVariants check for multiple traces). Closing as `blocked` rather than `complete`, since this differs from task-01's situation (task-01's Kill Criteria explicitly permitted an escalation-without-fix as a valid close condition; task-02 has no equivalent explicit allowance, so `blocked` is the honest status rather than overstating this as resolved).
