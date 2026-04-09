
# Sync Master Plan

## Context Required
MEDIUM-CONTEXT: Current plan overview.md (for Master Plan field), hub access

## Triggers

### Automatic
- Called by `execute-task` Step 9 on task completion (when Master Plan field is set)

### Manual
- `/sync-master-plan`
- `/sync-master-plan --all`
- "sync master plan"
- "update master plan status"
- "push status to hub"

## Instructions

### Step 0: Load Config and Resolve Hub

Read `.ai-framework.config` from the project root. Extract:
- `PLANS_DIR` — where plans live (default `.plans`)
- `LAYOUT` — `single` or `container`
- `WORK_PLANS_PATH` — explicit hub path (optional)
- `WORK_PLANS_BRANCH` — branch for hub git operations (default `main`)

Determine **sync mode** from invocation context:
- **`automatic`** — called by execute-task. Failures MUST be silently skipped (log warning, never block).
- **`manual`** — called by user via `/sync-master-plan`. Failures stop and report to user.

Resolve `HUB_PATH` and hub type (`external` or `self-hosted`). See [hub-resolution](references/hub-resolution.md) for the 4-tier priority lookup.

If `HUB_PATH` cannot be resolved:
- **Automatic mode**: log warning, skip sync, return success
- **Manual mode**: inform user "Hub not configured. Set WORK_PLANS_PATH in .ai-framework.config or run `/create-master-plan` to set up." and stop

---

### Step 1: Identify Master Plan Link

**If invoked with `--all`**: Scan all `{PLANS_DIR}/*/overview.md` files. For each, extract the `Master Plan` field. Build a list of plans linked to a master plan (field is not "None"). If no linked plans found, inform user and stop.

**If invoked from execute-task**: The master plan slug and plan-slug are passed directly — proceed to Step 2.

**Otherwise (single plan)**: If one plan directory exists, use it. If multiple, ask: "Which plan do you want to sync?" Read `{PLANS_DIR}/{plan-slug}/overview.md` and extract the `Master Plan` field.

- If field is "None" or absent → inform user "No master plan linked" and stop
- If field has a slug value → continue with that as the master plan slug

---

### Step 2: Resolve Repo Key

Determine the canonical repo key (used for `repos/{repo-key}.md` in the hub):

1. **Git remote**: `git remote get-url origin` → parse the repository name (strip `.git`, take last path segment)
2. **Fallback**: `AGENTS.md` project name field
3. **Fallback**: `basename` of the current working directory

---

### Step 3: Locate Master Plan in Hub

Scan the hub's state directories to find the master plan:

```
for state_dir in backlog planned active completed deprioritized cancelled; do
  if [ -d "$HUB_PATH/plans/$state_dir/$master_slug" ]; then
    MASTER_STATE="$state_dir"
    break
  fi
done
```

For **self-hosted** hubs: look for `{PLANS_DIR}/{master_slug}/overview.md` directly. If found, set `MASTER_STATE` to the plan's `Status` field value.

If not found: automatic mode → log warning, skip; manual mode → stop with error.
If found in `deprioritized` or `cancelled`: warn and continue.

---

### Step 4: Gather Local Progress

**If called from execute-task**: Update the specific task row keyed by `{task-path}` and refresh aggregate progress.

**If called manually**: Read all `status.md` files under `{PLANS_DIR}/{plan-slug}/`. Compile: total tasks, completed, in-progress, blocked, current work description.

**If called with --all**: For each linked plan, read `status.md` files and sync separately. Report: "{N} plans synced, {M} skipped (no master plan link)".

**Contract amendment scanning** (all modes): Scan each `status.md` Adaptations section for markers: `- contract-amendment: {contract-name} | {recommendation} | {task-path}`. Free-form prose is NOT sufficient — only exact marker lines are parsed.

---

### Step 5: Update Hub Files

Follow [hub-file-updates](references/hub-file-updates.md) for the detailed update procedure covering:
- `repos/{repo-key}.md` — progress summary, task status table, current work, blockers
- Contract amendments — upsert rows using `{contract-name} + {task-path}` as key
- `STATUS.md` — per-repo progress row, overall totals, timeline entry

---

### Step 6: Commit and Push

Follow [hub-commit-push](references/hub-commit-push.md) for the commit/push procedure covering self-hosted and external hubs, conflict resolution, and auto-skip behavior.

**Critical**: Sync failure must NEVER block task completion in the source repo.

---

### Step 7: Report Result

- **Success**: "Synced {repo-key} status to master plan {master_slug} ({done}/{total} tasks complete)"
- **Contract amendments found**: "Contract amendments recommended — review: {contract-names}"
- **Failure (automatic)**: "Sync to master plan skipped ({reason}). Task completion is not affected."
- **Failure (manual)**: "Sync failed: {reason}. Hub may need manual attention."
- **--all mode**: "{N} plans synced, {M} skipped. Contract amendments: {list or none}"

## Output

- Updated `repos/{repo-key}.md` and `STATUS.md` in hub (state directory resolved dynamically)
- Contract amendment rows upserted (if any found)
- Sync summary reported
- No blocking errors propagated to calling task

## Examples

**Automatic** (called by execute-task): Resolves hub → finds master plan in `plans/{state}/` → updates `repos/{repo-key}.md` (task row + progress) → updates `STATUS.md` → commits and pushes → returns.

**Manual single plan**: Reads all `status.md` files → compiles full progress → syncs to hub → reports status summary with current work and blockers.

**Container self-hosted**: Hub resolves to `{PLANS_DIR}/` → updates files locally → commits (no push needed).

**Sync all** (`--all`): Scans all plans for master plan links → syncs each linked plan separately → reports aggregate.
