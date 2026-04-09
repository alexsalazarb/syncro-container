---
name: execute-plan
description: >-
  Executes remaining tasks in a development plan with parallel agent support.
  Reads plan state, builds a dependency DAG, identifies ready tasks, and runs
  them in waves. Use when you want to run all remaining tasks in a plan.
---

# Execute Plan

## Context Required
HIGH-CONTEXT: Full plan directory, AGENTS.md

## Triggers

### Automatic
- None (manual only)

### Manual
- `/execute-plan {plan-slug}`
- "run the plan"
- "execute remaining tasks"
- "execute the {plan} plan"

## Instructions

### Step 0: Load Project Config

Read `.ai-framework.config` per [`common/config-defaults.md`](../common/config-defaults.md). Extract: `PLANS_DIR`, `PLANS_MODE`, `LAYOUT`, `PROJECT_DIR`.

Also extract (skill-specific):
- `PR_INTEGRATION` — default `false`
- `AUTO_MERGE_TASK_PRS` — default `false`
- `DEFAULT_BASE_BRANCH` — default `main`

**If `PR_INTEGRATION=true`**:
1. Verify `gh` CLI is available: `gh --version`. If not found, print install instructions and abort:
   ```
   Error: gh CLI is required for PR_INTEGRATION=true.
   Install: https://cli.github.com/
   Authenticate: gh auth login
   ```
2. Prompt the developer: "Base branch for this plan's final PR? (default: {DEFAULT_BASE_BRANCH})"
   Accept empty input as the default value.
3. Record as `BASE_BRANCH={confirmed value}`.

---

### Step 0b: Create Integration Branch (PR_INTEGRATION=true only)

Skip this step entirely when `PR_INTEGRATION=false`.

1. Create (or switch to) the plan integration branch from the confirmed base branch:
   ```bash
   git fetch origin
   if git show-ref --verify --quiet refs/heads/plan/{slug}; then
     git switch plan/{slug}
   else
     git switch -c plan/{slug} origin/{BASE_BRANCH}
     git push -u origin plan/{slug}
   fi
   ```
2. Write to `{PLANS_DIR}/{slug}/overview.md`:
   - Set `**Integration Branch**` to `plan/{slug}`
   - Set `**Base Branch**` to `{BASE_BRANCH}`
3. Commit: `plan({slug}): create integration branch plan/{slug} from {BASE_BRANCH}`

---

### Step 1: Load Plan State

1. Read `{PLANS_DIR}/{plan-slug}/overview.md` for phases and dependencies
2. Read ALL `status.md` files to build current state map
3. Categorize each task: `complete`/`adapted` (skip), `in-progress` (monitor), `blocked` (wait), `ready` (execute)
4. Detect layout: flat (`task-NN-slug/`) or phased (`phase-N/task-NN-slug/`)
5. **Master Plan lifecycle check** — if `overview.md` has `**Master Plan**` set (not "None"):
   1. Resolve hub path (config → git global → container self-host)
   2. Scan hub state dirs for the master plan slug
   3. If found in `deprioritized` or `cancelled`: warn the user:
      "Master plan '{slug}' is {state} in the hub. Proceeding may produce orphaned work."
      Ask: "Continue anyway? (y/n)"
   4. If hub is unavailable: log info and proceed (hub is optional for execution)

A task is **ready** when:
- Status is `not-started` or `blocked` (with blockers now resolved)
- All tasks in "Depends On" are `complete` or `adapted`

---

### Step 2: Build Execution Waves

Group ready tasks into waves based on the dependency DAG:

```text
Execution Plan for: {plan-slug}
================================
Completed: task-01 (skipping)
Wave 1: task-02, task-03 (parallel — no shared files)
Wave 2: task-04 (depends on Wave 1)
Blocked: task-05 (external dependency)
================================
Proceed? (y/n)
```

---

### Step 3: Execute Wave (With Agent Teams — Preferred)

For each wave with multiple ready tasks:
1. Spawn one implementer agent per task using the `Task` tool
2. Each agent runs `execute-task` for its assigned task in its own worktree (`isolation: "worktree"`)
3. Wait for all agents in the wave to complete

---

### Step 4: Execute Wave (Sequential Fallback)

If agent teams are not available:
1. Execute each ready task one at a time via `execute-task`
2. After each task, re-read status files from `{PLANS_DIR}` to check for newly unblocked tasks

---

### Step 5: Handle Blockers

If a task is blocked:
- Present blocker details to the user
- Offer: skip (mark as adapted), resolve manually, or abort plan

---

### Step 6: Phase Gate Verification

After each wave/phase completes, verify before proceeding:
1. All Phase N tasks are `complete` or `adapted`
2. Required tests were added and pass
3. No critical test regressions
4. Required KB / documentation updates are complete or explicitly tracked

**If `PR_INTEGRATION=true`**: Monitor task PRs targeting `plan/{slug}`:
1. List open PRs:
   ```bash
   gh pr list --base plan/{slug} --state open
   ```
2. If `AUTO_MERGE_TASK_PRS=true`: merge each open PR:
   ```bash
   gh pr merge {PR_NUMBER} --merge --delete-branch
   ```
3. If `AUTO_MERGE_TASK_PRS=false`: display the list and ask:
   "Please merge the above task PRs into `plan/{slug}` before proceeding. Continue? (y/n)"

---

### Step 7: Plan Completion

When all tasks are `complete` or `adapted`:
1. Update `{PLANS_DIR}/{plan-slug}/overview.md` status to `complete`
2. Move plan row from Active to Completed table in `{PLANS_DIR}/README.md`
3. If `PLANS_MODE=repo`: remind user to commit and push the plans repo
4. Report summary: tasks completed vs adapted vs blocked, branches created, next steps
5. Suggest: `/archive-plan {plan-slug}`

**If `PR_INTEGRATION=true`** (after all task PRs are merged into `plan/{slug}`):

6. Open the final plan PR from `plan/{slug}` to `{BASE_BRANCH}`:
   ```bash
   gh pr create \
     --title "plan({slug}): merge integration branch into {BASE_BRANCH}" \
     --body "## Plan: {plan title}

   {objective from overview.md}

   ### Tasks completed
   {task summary list from overview.md}" \
     --base {BASE_BRANCH} \
     --head plan/{slug}
   ```
7. Print the PR URL.
8. Remind the developer: "This is the final human-reviewed PR. Review, approve, and merge to complete the plan."

## Output

- All plan tasks executed (or blocked with explanations)
- Status files updated for every task in `{PLANS_DIR}`
- Phase gate verifications passed
- Plan overview and index updated
- Summary report
