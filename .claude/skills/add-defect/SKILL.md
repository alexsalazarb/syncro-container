---
name: add-defect
description: >-
  Adds a defect task to an existing plan. Use when QA, a reviewer, or a developer
  finds a bug in a plan's deliverables that the plan must fix before it can ship.
  Creates the defect task directory, updates the plan overview Defects table and
  Task Summary, and commits to main. The defect can optionally block an existing task.
---

# Add Defect

## Context Required
MEDIUM-CONTEXT: Plan overview, existing task list, bug description from user

## Triggers

### Automatic
- None (manual only)

### Manual
- `/add-defect {plan-slug} "{bug description}"`
- `/add-defect {plan-slug} --blocks {task-path} "{bug description}"`
- `/add-defect {plan-slug} {JIRA-KEY}`
- "add a bug to the plan"
- "found a bug in {plan-slug}"
- "QA found an issue with..."
- "this plan has a defect"

## When to Use This vs /create-bug-plan

| Situation | Use |
|-----------|-----|
| Bug is in code that THIS PLAN delivered or changed | `/add-defect` — the plan owns it |
| Bug blocks a task in this plan from completing | `/add-defect --blocks {task-path}` |
| Bug is a production issue unrelated to any active plan | `/create-bug-plan` |
| Bug needs deep investigation (root cause unclear) | `/create-bug-plan` |
| Bug is trivial (1 file, obvious fix, <30 min) | Just fix it — no plan or defect needed |

## Instructions

### Step 1: Load Plan Context

1. Read `.plans/{plan-slug}/overview.md` for the plan's current state
2. Read the Task Summary table to understand existing tasks and numbering
3. If a JIRA key was provided, fetch the ticket for details

### Step 2: Gather Bug Details

If not provided as arguments, ask for:
- **What's broken**: Clear description of the bug
- **Where it was found**: Which task's deliverables, which feature area, repro steps
- **Blocks**: Does this block an existing task? Which one?
- **Severity**: P0-P3 (use same scale as create-bug-plan Step 1b)
- **JIRA**: Ticket key if one exists (or should one be created?)

### Step 3: Create Defect Task

1. **Assign the next task number** — Find the highest `task-{NN}` number across all tasks in the plan (including any phase subdirectories) and increment.

2. **Generate the task slug**: `task-{NN}-defect-{2-3-word-description}`

3. **Determine placement**:
   - Flat plan (no phases): Place in plan root
   - Phased plan: Place in the same phase as the task it blocks, or the latest phase if it doesn't block anything

4. **Create the task directory**:
   ```bash
   mkdir -p .plans/{plan-slug}/{task-path}
   ```

5. **Write `task.md`** from the defect task template (see below)

6. **Write `status.md`** — Standard status template, set to `not-started`

### Step 4: Update Plan Overview

1. **Task Summary table** — Add the defect task row:
   ```markdown
   | {task-path} | Defect: {title} | {phase} | not-started | {blocked-task or "—"} |
   ```

2. **Defects table** — Add an entry:
   ```markdown
   | {task-path} | {title} | {found-during or "QA"} | {blocks or "—"} | not-started |
   ```

3. **If the defect blocks an existing task**:
   - Read that task's `status.md`
   - If the task is `in-progress`, add a blocker entry: "Blocked on {defect-task-path}: {description}"
   - If the task is `complete` or `adapted`, the defect is a regression — note this in the defect task context

### Step 5: Git Operations

Plan metadata goes directly to main:

```bash
git checkout main && git pull
git add .plans/{plan-slug}/{task-path}/task.md .plans/{plan-slug}/{task-path}/status.md .plans/{plan-slug}/overview.md
git commit -m "plan({plan-slug}): add defect {task-path}"
git push origin main
```

If the defect blocks a task whose status.md was updated:
```bash
git add .plans/{plan-slug}/{blocked-task-path}/status.md
```

### Step 6: Report

Present to the user:
- Defect task path and title
- What it blocks (if anything)
- Severity
- Suggest next step: `/execute-task {plan-slug}/{defect-task-path}`

## Defect Task Template

```markdown
# Task: Defect — [TITLE]

**Plan**: [Plan name]
**Task ID**: task-[NN]
**Task Path**: task-[NN]-defect-[slug]
**Depends On**: "None"
**Blocks**: [task-path] or "None"
**JIRA**: [JIRA-KEY] or "N/A"
**Severity**: P0 | P1 | P2 | P3
**Found By**: [name or "QA"]
**Found During**: [task-path, "QA testing", or "code review"]

## Bug Description

[What's broken, how to reproduce, expected vs actual behavior]

## Root Cause

<!-- Brief if known, "TBD — needs investigation" if not.
     If this needs deep investigation, consider /create-bug-plan instead. -->

**File**: [path] (lines [N-M]) or "TBD"
**What it does**: [current behavior]
**What it should do**: [expected behavior]

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| [path] | modify | [Brief note] |

### Do NOT Modify

- [Files owned by sibling tasks]

## Implementation Steps

### Step 1: Fix
[Specific fix instructions, or investigation steps if root cause is TBD]

### Step 2: Regression Test
[Test that fails before fix, passes after]

## Testing

- [ ] Regression test added (fails before, passes after)
- [ ] Existing tests still pass
- [ ] `pre-commit-check` passes
- [ ] Original repro steps no longer reproduce the bug

## Completion Criteria

- [ ] Bug fixed
- [ ] Regression test added
- [ ] Existing tests pass
- [ ] Changes committed to `plan/[slug]/[task-path]` branch
- [ ] Status updated in `status.md`
- [ ] Blocked task unblocked (if applicable — remove blocker from its status.md)
```

## Output

- `.plans/{plan-slug}/{task-path}/` directory with task.md and status.md
- Updated overview.md (Task Summary + Defects table)
- Updated blocked task status.md (if applicable)
- All changes committed to main

## Examples

### QA finds a bug in a feature plan

```
User: /add-defect calendar-performance-typescript "Date picker crashes when switching months in Firefox"

Agent:
1. Reads .plans/calendar-performance-typescript/overview.md
2. Finds highest task is task-06, assigns task-07
3. Creates task-07-defect-datepicker-firefox/
4. Updates overview Task Summary and Defects tables
5. Commits to main
6. Reports: "Added defect task-07-defect-datepicker-firefox to calendar-performance-typescript.
   Severity: P2. Doesn't block any existing task. Run /execute-task calendar-performance-typescript/task-07-defect-datepicker-firefox to fix."
```

### QA finds a regression that blocks a task

```
User: /add-defect ip-12998-time-entry-overlap --blocks task-03-verify-consumers "Tech Portal shows raw JSON error instead of user-friendly message"

Agent:
1. Reads the plan, finds task-03 is in-progress
2. Creates task-04-defect-techportal-error-display/
3. Adds blocker to task-03 status.md
4. Updates overview
5. Reports: "Added defect task-04-defect-techportal-error-display. Blocks task-03-verify-consumers
   (marked as blocked). Run /execute-task ip-12998-time-entry-overlap/task-04-defect-techportal-error-display to fix."
```
