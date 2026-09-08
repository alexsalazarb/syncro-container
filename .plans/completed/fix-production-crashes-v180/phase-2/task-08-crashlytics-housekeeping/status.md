# Status: Close 3 stale Crashlytics issues already fixed in code

**Current Status**: complete
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: — (no code, Crashlytics API only)
**PR**: N/A (PR_INTEGRATION=false)

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-09-07 | not-started | claude | Task created |
| 2026-09-07 | complete | claude | Re-verified all 3 still stale, closed all 3, left explanatory notes on each |

## Summary

Re-ran `crashlytics_get_report(topVersions)` for each of the 3 issues, scoped to the last 90 days: all showed **0 events on every recent version** (1.6.1, 1.7.0, 1.7.1) — the only non-zero counts were a handful of historical events on old `1.4.0 (412)` installs, which pre-date the fixes and don't indicate a current regression. No new events since 2026-09-07, so none needed escalating per the task's stop condition.

Closed all 3 via `crashlytics_update_issue` (state: CLOSED) and left a note on each via `crashlytics_create_note` documenting why and which plan fixed them, so future triage doesn't re-derive this.

| Issue ID | Closed | Note left |
|----------|--------|-----------|
| `9a1d349f325abb563d2b26653a1b993c` (asset_filter_deserializer) | ✅ | ✅ |
| `de94f83f3e5808649141d2c1d40c58a8` (WorksheetTemplateDeserializer) | ✅ | ✅ |
| `7ec1fff22d998a861ff0d1705d05518d` (DioMixin 504) | ✅ | ✅ |

## Verification

- [x] All 3 re-verified as still stale (0 events on recent versions) before closing
- [x] All 3 marked CLOSED in Crashlytics (confirmed via the `crashlytics_update_issue` response payload for each)

## Artifacts

- 3 Crashlytics `state: CLOSED` updates + 3 explanatory notes (see Summary table above for issue IDs)

## Adaptations

None — task.md's plan executed exactly as written.
