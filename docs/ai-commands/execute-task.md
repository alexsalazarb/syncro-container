
# Execute Task

## Context Required
HIGH-CONTEXT: Plan overview, task definition, dependency status files, AGENTS.md

## Triggers

### Automatic
- None (manual only)

### Manual
- `/execute-task {plan-slug}/{task-path}`
- `/execute-task {plan-slug}/task-01-data-model`
- `/execute-task {plan-slug}/phase-1/task-01-data-model`
- "work on {task-path} from {plan}"

## Instructions

### Step 0: Load Project Config

Read `.ai-framework.config` from the project root. Extract:
- `PLANS_DIR` — where plans live (e.g., `.plans`)
- `PLANS_MODE` — `local` or `repo`
- `LAYOUT` — `single` or `container`
- `PROJECTS` (container) — comma-separated `folder:stack` pairs, e.g. `myapp:swift,backend:rails`
- `PROJECT_DIR` / `STACK` (single) — single project folder and stack

If missing, use defaults: `PLANS_DIR=.plans`, `PLANS_MODE=local`, `LAYOUT=single`, `PROJECT_DIR=.`

Also extract `PR_INTEGRATION` (default `false`). If `true`, read `**Integration Branch**` from `{PLANS_DIR}/{plan-slug}/overview.md`; abort if missing or `N/A`.

**Resolve active project** (follow [`common/resolve-project-context.md`](../common/resolve-project-context.md)):
1. Build the `folder→stack` map from `PROJECTS` (or `{PROJECT_DIR: STACK}` for single layout).
2. Infer the active project from the task path — check which folder its files touch, or the plan's `overview.md` `Project:` metadata.
3. If still ambiguous, ask: "Which project? (`{folder1}` / `{folder2}` / ...)"

Resolved variables: `PROJECT_CODE_ROOT`, `PROJECT_CONTEXT_ROOT`, `STACK`, `AGENTS_MD` — see [`common/resolve-project-context.md`](../common/resolve-project-context.md).

---

### Step 1: Load Task Context

Task dirs may be flat (`task-XX-{slug}`) or phased (`phase-N/task-XX-{slug}`).

1. Read `{PLANS_DIR}/{plan-slug}/overview.md` for plan context
2. Read `{PLANS_DIR}/{plan-slug}/{task-path}/task.md` for task definition
3. Read `{PLANS_DIR}/{plan-slug}/{task-path}/status.md` for current state
4. **If `investigation.md` exists** in the task directory — this is a bug plan. Read it for root cause context, severity, and test gap analysis.

If the task is already `complete`, inform the user and stop.
If `in-progress`, ask the user if they want to resume or restart.

---

### Step 2: Pre-Flight Protocol

1. Pull latest on the source branch:
   - **If `PR_INTEGRATION=false`**: `git switch main && git pull --rebase origin main`
   - **If `PR_INTEGRATION=true`**: `git fetch origin && git switch plan/{slug} && git pull --rebase origin plan/{slug}`
2. **Read JIRA**: If `task.md` has a JIRA field that is not "N/A", read the ticket description AND comments for latest requirements
3. Check prerequisites: verify each task in "Depends On" has `status.md` showing `complete` or `adapted`
4. Read referenced KB docs and architecture files mentioned in `task.md`
5. Review existing tests for the affected area

---

### Step 3: Create Git Branch

Follow [task-branch-creation](references/task-branch-creation.md) for the branch commands (handles both `PR_INTEGRATION` modes).

---

### Step 4: Check File Ownership

Before modifying any file, verify it is in the File Ownership table in `task.md`.
If a needed file is listed in **Do NOT Modify**, stub the dependency and note it in `status.md`.

---

### Step 5: Execute Implementation Steps

Follow steps in `task.md`. When contradictions are found, apply this priority:
1. Architecture docs / KB entries — highest authority
2. Task file — plan-time assumptions (may be stale)

**When to adapt** (handle within the task):
- Different implementation path achieves the same outcome
- Additional minor requirement discovered within scope

**When to escalate** (mark blocked, stop work):
- Scope has fundamentally changed
- File ownership conflict with a parallel task
- Blocker requires re-planning

Document all adaptations in `status.md` Adaptations section.
Implement tests and any KB/doc updates called for in `task.md`.

---

### Step 5a: Bug Discovery Protocol

Follow [bug-discovery-protocol](references/bug-discovery-protocol.md) for triage rules and defect task creation steps.

---

### Step 6: Run Verification

1. Run linter (as defined in AGENTS.md for this project)
2. Run tests (as defined in AGENTS.md for this project)
3. Run `pre-commit-check` on changed files
4. Verify completion criteria from `task.md`
5. KB/doc review: update existing docs or run `document-solution` if needed

**Bug plan extra** (if `investigation.md` exists): Verify regression test fails pre-fix and passes after; confirm fix matches root cause in `investigation.md`; check consumer impact for affected systems.

---

### Step 6b: Open Task PR (PR_INTEGRATION=true only)

Follow [task-pr-steps](references/task-pr-steps.md) for PR creation commands. Skip entirely when `PR_INTEGRATION=false`.

---

### Step 7: Update Status and Plan Index

Update `{PLANS_DIR}/{plan-slug}/{task-path}/status.md`:
- Set status to `complete` (or `adapted` if deviations were made)
- Add completion timestamp
- Record branch and PR link (set PR to "N/A" when `PR_INTEGRATION=false`)

Update `{PLANS_DIR}/{plan-slug}/overview.md` task summary table.
Update progress count in `{PLANS_DIR}/README.md`.

If `PLANS_MODE=repo`: commit plan changes in the separate plans repo.
If `PLANS_MODE=local`: include plan file changes in the same commit or a follow-up.

---

### Step 7b: Sync Master Plan to Hub (automatic)

Skip if `overview.md` has no `**Master Plan**` field or value is "None".

Run `sync-master-plan` in automatic mode (pass plan-slug, task-path, new-status). If sync fails, log warning in `status.md` Adaptations section and continue — sync failure must never block task completion.

---

### Step 8: Commit Code

```bash
git add [owned files]
git commit -m "plan({plan-slug}): {task title}"
git push -u origin plan/{plan-slug}/{task-path}
```

---

### Step 9: Sync Master Plan (Conditional)

Skip if `overview.md` has no `Master Plan` field or value is "None".

Read `{PLANS_DIR}/{master-plan-slug}/overview.md` and update the row for this local plan to reflect the completed task. If all linked local plans are complete/adapted, mark the master plan as complete and ready for archival. If update fails, report warning only — do NOT block task completion.

---

### Step 9a: Bug Plan Completion Ceremony (Conditional)

**Trigger**: `investigation.md` exists AND all sibling tasks are `complete` or `adapted`.

Follow `references/bug-plan-completion.md` for the ceremony steps (success criteria verification, plan status update, work-plans sync, Jira transition, and staging verification prompt).

---

### Step 10: Report Completion

Present: task status (completed/adapted), branch name, files changed, test results, KB update outcome, bug plan completion (if applicable), newly unblocked tasks, and suggested next task.

## Output

- Implemented changes on `plan/{plan-slug}/{task-path}` branch
- Updated `status.md` in `{PLANS_DIR}` with completion details
- Updated plan overview, plans index, and master plan (if linked)
- Bug plan completion ceremony run (if applicable)
- Report of completed work and newly unblocked tasks
