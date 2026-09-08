# Task: Close 3 stale Crashlytics issues already fixed in code

**Plan**: fix-production-crashes-v180
**Phase**: 2
**Task ID**: task-08
**Task Path**: `phase-2/task-08-crashlytics-housekeeping`
**Depends On**: None
**Ticket**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)

## Objective

Mark 3 Crashlytics issues as CLOSED — they are already fixed in code (verified via git history) or explicitly out of scope, but the console still shows them OPEN, which creates false-positive noise in future triage. No code change.

## Context

See [investigation.md](../investigation.md) Task 3. `crashlytics_get_report` (90-day topIssues) shows all 3 issues' `lastSeenVersion` is 2-4 releases behind current (1.8.0) — no recurrence since their fixes shipped.

## Before You Start

- [ ] Verify every prerequisite task in `Depends On` is complete (None — no dependencies)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

None — this task only calls the Firebase MCP `crashlytics_update_issue` tool, no repo files change.

## Implementation Steps

### Step 1: Re-verify each issue is still stale before closing
Re-run `crashlytics_get_report` (topIssues, last 90 days) for the Android app (`1:920223298498:android:3352304fd0baa59e5b5c5b`) and confirm these 3 issues still show no events since the dates recorded below. If any has NEW events since 2026-09-07, stop and treat it as a regression — do not close, escalate instead (it likely needs its own investigation, same as task-05 in this plan is a regression of SE-12758).

| Issue ID | Title | Last confirmed seen | Fixed by |
|----------|-------|---------------------|----------|
| `9a1d349f325abb563d2b26653a1b993c` | `asset_filter_deserializer` | v1.5.2 | SE-12498 (`fix-production-crashes-v152`) |
| `de94f83f3e5808649141d2c1d40c58a8` | `WorksheetTemplateDeserializer` | v1.4.0 | SE-12500 (`fix-production-crashes-v152`) |
| `7ec1fff22d998a861ff0d1705d05518d` | `DioMixin` 504 Gateway Timeout | v1.5.2 | N/A — backend-owned, explicitly out of scope |

### Step 2: Close each issue
Call `mcp__firebase__crashlytics_update_issue` with `appId: "1:920223298498:android:3352304fd0baa59e5b5c5b"`, the issue's `issueId`, and `state: "CLOSED"` for each of the 3 issues above.

### Step 3: Leave a note on each closed issue (optional but recommended)
Use `crashlytics_create_note` to record why it was closed and which plan fixed it, so future triage doesn't need to re-derive this. Example note: `"Closed 2026-09-07 — fixed in SE-12498 (fix-production-crashes-v152), no recurrence since v1.5.2."`

## Testing

- [ ] N/A — no code change, no automated test applicable
- [ ] Manual verification: confirm each issue shows `state: CLOSED` via a follow-up `crashlytics_get_issue` call

## Documentation / KB Updates

- [ ] No KB/doc updates required

## Completion Criteria

- [ ] All 3 issues re-verified as still stale (no new events) before closing
- [ ] All 3 issues marked CLOSED in Crashlytics
- [ ] Status updated in `status.md`
