# Hub File Updates

## 5a: Update `repos/{repo-key}.md`

For **external** hub: edit `{HUB_PATH}/plans/{MASTER_STATE}/{master_slug}/repos/{repo-key}.md`
For **self-hosted** hub: edit `{PLANS_DIR}/{master_slug}/repos/{repo-key}.md` (create `repos/` dir if needed)

If the file does not exist, create it from `templates/master-plan-repo-status.md`.

Update these sections:
- **Progress Summary**: Done/Total, In Progress, Blocked counts
- **Task Status** table: upsert each task row by task-path (status, branch, PR, notes)
- **Current Work**: list tasks with `in-progress` status
- **Blockers**: list tasks with `blocked` status and their blocker descriptions
- **Updated** date and **Updated By** field
- **Phase**: current phase (highest phase with in-progress tasks, or latest completed)

## 5b: Update Contract Amendments

If contract amendments were found in Step 4:
- In `repos/{repo-key}.md`, upsert rows in the **Contract Amendments** table
- Use `{contract-name} + {task-path}` as the stable key: if a row with that key exists, replace it; otherwise append
- Set Date to today, Status to "recommended"
- If `{HUB_PATH}/plans/{MASTER_STATE}/{master_slug}/contracts/` exists (external hub), also stage the repo file for commit

## 5c: Update `STATUS.md`

For **external** hub: edit `{HUB_PATH}/plans/{MASTER_STATE}/{master_slug}/STATUS.md`
For **self-hosted** hub: edit `{PLANS_DIR}/{master_slug}/STATUS.md` (create if needed)

Update:
- Per-repo progress row in the aggregate table (repo key, done/total, status, last synced)
- Overall totals across all repos
- Timeline entry: "{date}: {repo-key} synced ({done}/{total} complete)"
