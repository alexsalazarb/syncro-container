
# Archive Plan

## Context Required
MEDIUM-CONTEXT: Plan directory, plans README

## Triggers

### Automatic
- None (manual only)

### Manual
- `/archive-plan {plan-slug}`
- "archive the {plan} plan"
- "close out {plan}"

## Instructions

### Step 0: Load Project Config

Read `.ai-framework.config` per [`common/config-defaults.md`](../common/config-defaults.md). Extract: `PLANS_DIR`, `PLANS_MODE`.

---

### Step 1: Verify Completion

1. Read `{PLANS_DIR}/{plan-slug}/overview.md`
2. Check that every task shows status `complete` or `adapted` in the Task Summary table
3. Verify all Success Criteria are met
4. If any task is not complete, report the gap and stop unless user confirms `--force`

---

### Step 2: Move Plan to completed/

```bash
mkdir -p {PLANS_DIR}/completed
mv {PLANS_DIR}/{plan-slug} \
   {PLANS_DIR}/completed/{plan-slug}
```

---

### Step 3: Update Plans Index

Remove the plan row from **Active Plans** in `{PLANS_DIR}/README.md` and add to **Completed Plans**:

```markdown
## Completed Plans

| Plan | Objective | Tasks | Completed |
|------|-----------|-------|-----------|
| [Plan Title](completed/{slug}/overview.md) | Objective | N/N | YYYY-MM-DD |
```

---

### Step 4: Update Overview Status

Update `{PLANS_DIR}/completed/{plan-slug}/overview.md`:
- Set `**Status**: complete`
- Set `**Last Updated**: [TODAY]`

---

### Step 4b: Transition Hub Master Plan (If Linked)

Check `{PLANS_DIR}/completed/{plan-slug}/overview.md` for a `**Master Plan**` field.
If the value is "None" or the field is absent, skip this step.

If a master plan slug is set:

1. Resolve hub directory per [`common/hub-resolution.md`](../common/hub-resolution.md).
   If hub is unavailable (tier 4) or unreachable, log a warning
   ("Hub unavailable — skipping master plan transition check") and continue to Step 5.
2. Find the master plan in the hub's state directories (see hub-resolution.md § State Directories).
3. Read the hub's `STATUS.md` for the master plan. Check whether **all** repos listed
   in the status table show a completed state.
4. If **all** repos are complete: run `transition-plan` logic to move the master plan
   directory to `{HUB_DIR}/plans/completed/{master-plan-slug}/` and update its status.
5. If some repos are still in progress: log info
   ("Master plan has other repos still active; not transitioning to completed") and continue.

---

### Step 5: Commit and Report

If `PLANS_MODE=local`: commit the archive move with the code repo.
If `PLANS_MODE=repo`: commit and push the plans repo separately.

```text
Plan '{plan-slug}' archived.
- Moved to {PLANS_DIR}/completed/{plan-slug}/
- Updated {PLANS_DIR}/README.md
- Status: complete
```

## Output

- Plan directory moved to `{PLANS_DIR}/completed/{plan-slug}/`
- `{PLANS_DIR}/README.md` updated
- `overview.md` status set to `complete`
- Changes committed
