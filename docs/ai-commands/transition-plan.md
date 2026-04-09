
# Transition Plan

## Context Required
MEDIUM-CONTEXT: Plan slug, target state, work-plans hub access

## Triggers

### Automatic
- None (manual only)

### Manual
- `/transition-plan {slug} --to {state}`
- "move {plan} to active"
- "promote {plan} to active"
- "deprioritize {plan}"
- "cancel {plan}"
- "complete {plan}"
- "transition {plan} to {state}"

## Instructions

### Step 0: Load Config and Resolve Hub

> See [resolve-project-context](../common/resolve-project-context.md)

Read `.ai-framework.config` at the repository root. Extract: `LAYOUT`, `PLANS_DIR`, `WORK_PLANS_PATH`, `WORK_PLANS_BRANCH`.

Defaults: `PLANS_DIR=.plans`, `WORK_PLANS_BRANCH=main`.

Resolve the hub location using the [hub-priority-table](references/hub-priority-table.md). Store as `HUB_ROOT` (mode: `external` or `self-hosted`).

Verify `HUB_ROOT` exists and contains `plans/INDEX.md`.

---

### Step 1: Parse Arguments

Required:
- `{slug}`: Plan slug (kebab-case identifier)
- `--to {state}`: Target state — one of: `backlog`, `planned`, `active`, `completed`, `deprioritized`, `cancelled`

Optional:
- `--reason "..."`: Explanation for the transition. **Required** when target is `deprioritized` or `cancelled`. If not provided for those targets, prompt the user.

---

### Step 2: Find Plan Current Location

Scan all 6 state directories to locate the plan:

```bash
for state_dir in backlog planned active completed deprioritized cancelled; do
  if [ -d "$HUB_ROOT/plans/$state_dir/$slug" ]; then
    CURRENT_STATE="$state_dir"
    break
  fi
done
```

If the plan is not found in any state directory, report the error and stop.

---

### Step 3: Validate Transition

See [state-transitions](references/state-transitions.md) for the state machine and validation rules.

---

### Step 4: Move Plan Directory

**External hub** — fetch and rebase first:

```bash
cd "$HUB_ROOT"
git fetch origin "$WORK_PLANS_BRANCH"
git switch "$WORK_PLANS_BRANCH"
git rebase "origin/$WORK_PLANS_BRANCH"
git mv "plans/$CURRENT_STATE/$slug" "plans/$TARGET_STATE/$slug"
```

**Self-hosted hub**:

```bash
git mv "$HUB_ROOT/plans/$CURRENT_STATE/$slug" "$HUB_ROOT/plans/$TARGET_STATE/$slug"
```

---

### Step 5: Update plan.md Status Field

Read `plans/$TARGET_STATE/$slug/plan.md` and update the `**Status**:` field:

| State | Status value |
|-------|-------------|
| backlog | `backlog` |
| planned | `planned` |
| active | `in-progress` |
| completed | `complete` |
| deprioritized | `deprioritized` |
| cancelled | `cancelled` |

---

### Step 6: Update STATUS.md

Append to the Timeline table in `plans/$TARGET_STATE/$slug/STATUS.md`:

```markdown
| {YYYY-MM-DD} | Plan transitioned from {CURRENT_STATE} to {TARGET_STATE}{reason_suffix} |
```

Where `{reason_suffix}` is ` — {reason}` if a reason was provided, empty otherwise.

**If → `active`**: add `**Health**: on-track` to STATUS.md if not present.
**If → `planned`**: add `**Priority**: normal` to plan.md if not present.

---

### Step 7: Record Reason for Deprioritized / Cancelled

If `TARGET_STATE` is `deprioritized` or `cancelled`, append to the **Key Decisions** table in `plan.md`:

```markdown
| {YYYY-MM-DD} | Plan {TARGET_STATE} | {reason} |
```

---

### Step 8: Update INDEX.md

Follow [index-section-formats](references/index-section-formats.md) for the column format for each state section.

1. **Remove** the plan's row from the section matching `CURRENT_STATE`
2. **Add** a new row to the section matching `TARGET_STATE`

Read field values from `plan.md` and STATUS.md. Use today's date for date columns.

---

### Step 9: Commit and Push

**External hub:**

```bash
cd "$HUB_ROOT"
git add "plans/$TARGET_STATE/$slug/" plans/INDEX.md
git add -u "plans/$CURRENT_STATE/$slug"
git commit -m "chore: transition $slug from $CURRENT_STATE to $TARGET_STATE"
git push origin "$WORK_PLANS_BRANCH"
```

**Self-hosted hub:**

```bash
git add "$HUB_ROOT/plans/$TARGET_STATE/$slug/" "$HUB_ROOT/plans/INDEX.md"
git add -u "$HUB_ROOT/plans/$CURRENT_STATE/$slug"
git commit -m "chore: transition $slug from $CURRENT_STATE to $TARGET_STATE"
```

No push for self-hosted — the commit stays local until the next project push.

---

### Step 10: Report

Report: plan slug and title, transition (`{CURRENT_STATE}` → `{TARGET_STATE}`), reason (if provided), updated files, and hub mode.

## Output

- Plan directory moved from `plans/{CURRENT_STATE}/{slug}/` to `plans/{TARGET_STATE}/{slug}/`
- plan.md Status field updated
- STATUS.md timeline entry appended (+ Health if → active)
- plan.md Priority added if → planned; Key Decisions updated if → deprioritized/cancelled
- INDEX.md row moved between sections

## Examples

```
/transition-plan calendar-performance --to active
→ Moves from planned/ to active/, sets Status: in-progress, adds Health: on-track

/transition-plan inspection-scheduling --to deprioritized --reason "Blocked on Path A/B decision"
→ Moves from active/ to deprioritized/, records reason in Key Decisions

/transition-plan user-auth-flow --to completed
→ Moves from active/ to completed/, sets Status: complete
```
