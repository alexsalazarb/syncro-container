# Bug Plan Completion Ceremony

Run when `investigation.md` exists AND all sibling tasks are `complete` or `adapted`.

## Step 1: Verify Success Criteria

Read each checkbox in `overview.md` → Success Criteria:
- Verifiable from task outputs → mark checked
- Requires manual verification (e.g., staging) → leave unchecked, note as pending
- Any automated criterion fails → set plan status to `in-progress` with blocker; do NOT mark complete

## Step 2: Verify Completion Checklist

Same logic as Step 1. Cross-reference "Investigation.md root cause matches actual fix" item against what was actually changed.

## Step 3: Update Plan Status

If all automatable criteria pass:
- Set `overview.md` status to `complete` and `Last Updated` to current date
- Update `{PLANS_DIR}/README.md` to show all tasks complete

## Step 4: Sync Master Plan (If Master Plan Exists)

Skip this step if `overview.md` has no `Master Plan` field or value is "None".

The master plan lives locally at `{PLANS_DIR}/{master-plan-slug}/overview.md`.
Update the master plan's progress to reflect that this local bug plan is complete.
If all local plans linked to the master plan are complete, update the master plan status to `complete` and note it is ready for archival via `/archive-plan`.

## Step 5: Jira Transition (If JIRA Epic Set)

Skip this step if `overview.md` `JIRA Epic` is "N/A" or not present.

If Jira MCP available and JIRA key exists:
- Fetch transitions via `getTransitionsForJiraIssue`
- Transition to "Ready for QA" (or equivalent)
- Add comment: "All fix tasks complete. Regression tests passing. Ready for QA verification."
- If MCP unavailable, output the transition recommendation for manual action.

## Step 6: Prompt for Staging Verification

Always end with:
> "Bug plan `{slug}` is complete. All {N} tasks done. Before closing the Jira ticket, verify the fix in staging using the original reproduction steps from the investigation."
