
# Plans Status

## Context Required
MEDIUM-CONTEXT: Plans README, active plan status files, hub INDEX/STATUS (when available)

## Triggers

### Automatic
- None (manual only)

### Manual
- `/plans-status`
- "show plan status"
- "what plans are active"

## Instructions

> This skill is **read-only** — it does not create, modify, or delete any files.

### Step 0: Load Project Config

Read `.ai-framework.config` per [`common/config-defaults.md`](../common/config-defaults.md). Extract: `PLANS_DIR`, `LAYOUT`, `WORK_PLANS_PATH`, `WORK_PLANS_BRANCH`.

Resolve hub directory per [`common/hub-resolution.md`](../common/hub-resolution.md). Store as `HUB_ROOT` (or mark `HUB_AVAILABLE=false` on tier 4).

---

### Step 1: Read Local Plans Index

Read `{PLANS_DIR}/README.md`. Parse the markdown tables:
- **Active Plans** table → count rows (exclude header/separator)
- **Backlog Plans** table → count rows (if section exists)
- **Completed Plans** table → count rows

Record the plan slug and title for each **active** plan.

If `{PLANS_DIR}/README.md` does not exist, report "No local plans directory found" and stop.

---

### Step 2: Aggregate Local Plan Task Status

For each active local plan:

1. Find all `status.md` files under `{PLANS_DIR}/{plan-slug}/` (flat or phased layout — search recursively: `**/status.md`)
2. Read each `status.md` and extract the status value (one of: `not-started`, `in-progress`, `complete`, `blocked`, `adapted`)
3. Count totals per status category
4. Compute progress: completed + adapted tasks out of total tasks
5. Note any `blocked` tasks — record the task path and blocker description from `status.md`

---

### Step 3: Read Hub Plans Index (if available)

Skip if `HUB_AVAILABLE=false`.

Read `{HUB_ROOT}/plans/INDEX.md`. Parse the lifecycle state sections:
- **Active** — count rows
- **Planned** — count rows
- **Backlog** — count rows
- **Completed** — count rows (if present)

Record the slug for each **active** master plan.

If `{HUB_ROOT}/plans/INDEX.md` does not exist, set `HUB_AVAILABLE=false` and continue with local-only output.

---

### Step 4: Read Active Master Plan Status (if available)

Skip if `HUB_AVAILABLE=false` or no active master plans found.

For each active master plan slug:

1. Read `{HUB_ROOT}/plans/active/{slug}/STATUS.md`
2. Extract:
   - **Health** — on-track / at-risk / blocked
   - **Repo progress** — how many repos are progressing vs total
   - Any blockers or notes

---

### Step 5: Aggregate Blocked Tasks

Collect all blocked items from Steps 2 and 4:
- Local blocked tasks: `{plan-slug}/{task-path}: {blocker description}`
- Hub blocked master plans: `{master-slug}: {blocker description}`

---

### Step 6: Present Dashboard

Output the dashboard as formatted text. Adapt sections based on what data is available.

**Full output (hub available)**:

```
Plans Status Dashboard
======================
Local Plans:  {active} active | {backlog} backlog | {completed} completed
Hub Plans:    {active} active | {planned} planned | {backlog} backlog

Active Local Plans:
  {plan-slug} ({done}/{total} tasks, {in-progress} in-progress{, N blocked if any})
  ...

Active Master Plans:
  {master-slug} ({health}, {progressing}/{total} repos progressing)
  ...

Blocked Tasks:
  {plan-slug}/{task-path}: {blocker description}
  ...
======================
```

**Local-only output (hub unavailable)**:

```
Plans Status Dashboard
======================
Local Plans:  {active} active | {backlog} backlog | {completed} completed
Hub Plans:    not configured

Active Local Plans:
  {plan-slug} ({done}/{total} tasks, {in-progress} in-progress{, N blocked if any})
  ...

Blocked Tasks:
  {plan-slug}/{task-path}: {blocker description}
  ...
======================
```

If no blocked tasks exist, replace the Blocked Tasks section with:
```
Blocked Tasks:
  (none)
```

## Output

- Formatted text dashboard printed to the conversation (no files created or modified)
- Graceful degradation: hub sections omitted when hub is not configured
- All data sourced from existing plan files — no external calls
