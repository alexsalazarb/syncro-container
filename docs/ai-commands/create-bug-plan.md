
# Create Bug Plan

## Context Required
HIGH-CONTEXT: AGENTS.md, KB index, relevant codebase files, ticket (if available)

## When to Use This Skill

See [bug-triage-flowchart](references/bug-triage-flowchart.md) for the decision tree on when to use this skill vs inline fixes, defect tasks, or ticket logging — and the comparison table between `/create-plan` and `/create-bug-plan`.

**Related skills**:
- `/add-defect {plan-slug}` — QA or reviewers add a defect task to an existing plan
- `/execute-task` Step 5a — handles bugs discovered mid-implementation

## Triggers

### Automatic
- None (manual only)

### Manual
- `/create-bug-plan`
- `/create-bug-plan [ticket key or URL]`
- `/create-bug-plan [bug description]`
- "create a bug plan for..."
- "investigate and plan this bug"
- "plan this bug fix"

## Instructions

### Step 0: Load Project Config

```bash
cat .ai-framework.config
```

Extract: `PLANS_DIR`, `PLANS_MODE`, `LAYOUT`, `PROJECT_DIR`, `PROJECTS`.
If missing, use defaults: `PLANS_DIR=.plans`, `PLANS_MODE=local`, `LAYOUT=single`, `PROJECT_DIR=.`

**Resolve active project** (follow [`common/resolve-project-context.md`](../common/resolve-project-context.md)):

Resolved variables: `AGENTS_MD`, `FRAMEWORK_KB_DIR` (Layer 1), `CONTAINER_KB_DIR` (Layer 2), `KB_DIR` (Layer 3) — see [`common/resolve-project-context.md`](../common/resolve-project-context.md).

---

### Step 1: Gather Bug Report

**If a ticket key/URL is provided and Jira MCP is available:**
1. Fetch the ticket (`getJiraIssue`)
2. Read all comments for additional context and prior analysis
3. Extract: summary, reproduction steps, hypothesis, related tickets

**If a description is provided or no Jira access:**
1. Ask for: what's broken, who's affected, how to reproduce, any prior investigation

Gather:
- **Trigger source**: ticket key, customer name, monitoring alert, or internal discovery
- **Base branch**: Which branch to fix from (default: `main`)
- **Assigned Dev/QA**: Ask if not obvious from ticket

---

### Step 1b: Triage Severity

Follow [severity-classification](references/severity-classification.md) for the severity table and P0 shortcut rules.

---

### Step 2: Investigate Root Cause

This is the key differentiator from `/create-plan`. Follow [investigation-protocol](references/investigation-protocol.md) for the full investigation procedure including:

- **Step 2a**: Origin Analysis — trace how the bug was introduced via `git blame`/`git log`
- **Step 2b**: Feature History Trace — scan `.plans/` and build a timeline (skip for P0)
- **Step 2c**: Test Coverage Analysis — explain why existing tests missed this
- **Step 2d**: Cross-Project Assessment — detect if the fix spans multiple projects
- **Step 2e**: Confidence Assessment — rate High/Medium/Low and determine next action

**Present investigation findings to the user and ask for confirmation before proceeding.**

---

### Step 3: Generate Plan Slug

Format: `bugfix-{2-4-word-slug}` or `{ticket-key}-{2-4-word-slug}` if a ticket was provided.

Examples: `bugfix-login-timeout`, `jira-123-sync-overlap`

Use the same slug across all projects for consistency.

---

### Step 4: Design Fix Tasks

Bug fix plans have a standard task pattern. Always include the regression test task.

**Standard tasks:**

1. **Fix task** (always) — The focused code change
2. **Regression test task** (always, mandatory) — A test that fails before the fix and passes after
3. **Verify consumers task** (when fix changes API/contract behavior) — Confirm all consumers handle the changed behavior
4. **Data cleanup task** (when bug created bad data) — Script to fix corrupted data; requires review before running

**For cross-project bugs:** design tasks per project. Each project gets its own plan with tasks scoped to that project. Document inter-project dependencies as blockers in status.md (not task dependencies).

For each task, populate from `references/bug-task.md`. Include explicit file ownership and "Do NOT Modify" lists.

---

### Step 5: Create Directory Structure

Follow [bug-plan-structure](references/bug-plan-structure.md) for the directory layouts (single-project and container multi-project).

#### Step 5b: Create Master Plan Entry (cross-project only)

If Step 2d identified this as a cross-project bug and `LAYOUT=container`:

1. Resolve the hub path (same resolution order as `create-master-plan`):
   a. `.ai-framework.config` `WORK_PLANS_PATH` (explicit)
   b. `git config --global ai-framework.workplanspath` (user default)
   c. Container self-hosting: `{PLANS_DIR}/` is the hub (default for containers)
2. Run the `create-master-plan` flow (Type: Bug/Support, Initial state: `active`, repos from Step 2d, investigation summary)
3. Set the `Master Plan` field in each project's local `overview.md` to `{master-slug}`

---

### Step 6: Update Plans Index

Add the new plan(s) to `{PLANS_DIR}/README.md` Active Plans table:

```markdown
| [{Bug Title}]({slug}/overview.md) | {Bug summary} | 0/{N} | not-started | TBD |
```

For container layout with multiple projects, add a row per local plan.

---

### Step 7: Hand Off Plan for Review

The plan files are written and ready. Do NOT commit automatically — the developer must review the investigation findings and task breakdown first.

Tell the user:

```
Bug plan ready for review.

  Plan: {PLANS_DIR}/{slug}/
  Files: overview.md, investigation.md, task directories, README.md (index updated)

Review the investigation and tasks, then push when satisfied:

  git add {PLANS_DIR}/{slug}/ {PLANS_DIR}/README.md
  git commit -m "plan: create bug plan {slug}"
  git push origin main

Plans push directly to main — no PR needed.
```

---

### Step 8: Prepare Ticket Comment (If Applicable)

Follow [ticket-comment-template](references/ticket-comment-template.md) for the comment structure. If Jira MCP is available, post directly via `addCommentToJiraIssue`. Otherwise, include under `## Ticket Comment (ready to post)` in your report.

---

### Step 9: Report Summary

Present to the user:
- **Severity**: P0/P1/P2/P3 with one-line justification
- Root cause summary (1-2 sentences)
- Number of tasks and what each does
- Affected systems / projects
- **Cross-project status**: master plan created (if applicable)
- Key risk: what could go wrong with the fix
- Branch convention: `plan/{slug}/{task-path}` branches, merge to `main` via PR
- **Ticket comment / Pending git commands**: include if MCP or Bash unavailable
- Suggest next step: `/execute-task {slug}/task-01-...`

---

## Bug Plan Completion (triggered by execute-task)

When `execute-task` completes the last task in a bug plan, the completion ceremony runs automatically (see execute-task Step 9a):

1. **Verify Success Criteria** — Every criterion in `overview.md` checked
2. **Update Plan Status** — `overview.md` status → `complete`
3. **Sync Master Plan** — If linked, run `sync-master-plan`; trigger `transition-plan` if all repos complete
4. **Jira Transition** (if MCP available) — Transition ticket to "Ready for QA" or equivalent
5. **Staging Verification Prompt** — "Verify the fix in staging with the original repro steps before closing."

## Output

- `{PLANS_DIR}/{slug}/` with `overview.md`, `investigation.md`, and task directories
- Updated `{PLANS_DIR}/README.md`
- If cross-project: master plan in hub with lifecycle tracking
- Ticket comment (posted or included in report)
- Summary of plan structure
