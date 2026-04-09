# Hub Commit and Push

Helper concept — `auto_skip`: when `SYNC_MODE=automatic`, log a warning and return without blocking. When `manual`, report the error and stop.

## Self-hosted hub (container)

The hub IS the local `{PLANS_DIR}/` — no separate repo operations needed.

1. Stage the modified hub files (`repos/{repo-key}.md`, `STATUS.md`)
2. Commit: `chore(status): sync {repo-key} status for {master_slug}`
3. No push — these files live in the same repo

## External hub

1. Save current directory; switch to `HUB_PATH`
2. Check for clean working tree: `git status --porcelain`
   - If dirty → `auto_skip("Hub has uncommitted changes")`
3. Fetch and switch to `WORK_PLANS_BRANCH`:
   ```
   git fetch origin "${WORK_PLANS_BRANCH}" 2>/dev/null || true
   git switch "${WORK_PLANS_BRANCH}" || git switch -c "${WORK_PLANS_BRANCH}" "origin/${WORK_PLANS_BRANCH}"
   git rebase "origin/${WORK_PLANS_BRANCH}"
   ```
   If rebase fails → `auto_skip("Unable to rebase")`
4. Apply the file updates (Step 5 edits)
5. Stage and commit:
   ```
   git add plans/{MASTER_STATE}/{master_slug}/repos/{repo-key}.md \
           plans/{MASTER_STATE}/{master_slug}/STATUS.md
   ```
   If no changes → "Master plan already in sync" and return
   ```
   git commit -m "chore(status): sync {repo-key} status for {master_slug}"
   ```
6. Push:
   ```
   git push origin "${WORK_PLANS_BRANCH}"
   ```
7. If push rejected (non-fast-forward) → proceed to conflict resolution
8. If push fails for other reasons (auth, network) → `auto_skip("Push failed: {error}")`
9. Return to original directory

## Conflict Resolution

Only applies to **external** hub when push is rejected:

1. `git pull --rebase origin "${WORK_PLANS_BRANCH}"`
2. If rebase conflicts on the generated files, rebuild `repos/{repo-key}.md` and `STATUS.md` from fresh local data, stage them, then `git rebase --continue`
3. `git push origin "${WORK_PLANS_BRANCH}"`
4. If push still fails:
   - **Automatic mode**: log warning, return success (do NOT block)
   - **Manual mode**: report failure, suggest user resolve manually
