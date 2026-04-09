
# Create Plan

## Context Required
HIGH-CONTEXT: AGENTS.md, KB index, relevant codebase files

## Triggers

### Automatic
- None (manual only)

### Manual
- `/create-plan`
- `/create-plan [feature description]`
- "create a plan for..."
- "plan this feature"
- "break this into tasks"

## Instructions

### Step 0: Load Project Config

Read `.ai-framework.config` from the project root. Extract:
- `PLANS_DIR` — where plans live (e.g., `.plans`)
- `PLANS_MODE` — `local` or `repo`
- `LAYOUT` — `single` or `container`
- `PROJECTS` (container) — comma-separated `folder:stack` pairs, e.g. `myapp:swift,backend:rails`
- `PROJECT_DIR` / `STACK` (single) — single project folder and stack

If missing, use defaults: `PLANS_DIR=.plans`, `PLANS_MODE=local`, `LAYOUT=single`, `PROJECT_DIR=.`

**Resolve active project** (follow [`common/resolve-project-context.md`](../common/resolve-project-context.md)):
1. Build the `folder→stack` map from `PROJECTS` (or `{PROJECT_DIR: STACK}` for single layout).
2. Infer the active project from files mentioned in the feature description, or recently edited files. Check which folder prefix matches the map.
3. If ambiguous or this is a container with multiple projects, ask: "Which project is this plan for? (`{folder1} ({stack1})` / `{folder2} ({stack2})` / ...)"

Resolved variables used throughout this skill (see [`common/resolve-project-context.md`](../common/resolve-project-context.md)):
- **`PROJECT_CODE_ROOT`** — `{folder}/` (container) or project root (single); use in File Ownership for source paths
- **`PROJECT_CONTEXT_ROOT`** — Layer 3 `AGENTS.md` + KB (embedded vs isolated)
- **`STACK`** — from the map lookup (used to load the right AGENTS.md and KB)
- **`AGENTS_MD`** — `{PROJECT_CONTEXT_ROOT}AGENTS.md`
- **`FRAMEWORK_KB_DIR`** — `.ai-framework/knowledge-base/{STACK}` (Layer 1)
- **`CONTAINER_KB_DIR`** — `{CONTAINER_KB_ROOT}/` (Layer 2)
- **`KB_DIR`** — Layer 3 KB root per §4 of `resolve-project-context` (isolated: `{PROJECT_CONTEXT_ROOT}/`; embedded: `{folder}/docs/kb-project/`)

---

### Step 1: Gather Feature Description & Classify

If not provided as an argument, ask the user to describe the feature.

Gather:
- What needs to be built or changed
- Any constraints or requirements
- Desired outcomes
- Demo date: Target demo/release date (ask if not specified)
- Dev/QA pairing: Assigned developer and QA engineer (ask if not specified, "unassigned" is acceptable)
- Master plan linkage: `--master-plan {master-plan-slug}` argument, or ask if this local plan rolls up into a master plan at `{PLANS_DIR}/{master-plan-slug}/`

Classify the plan type:
- **Type 1 (Product)**: User-facing behavior change
- **Type 2 (Technical)**: Infrastructure, refactor, performance, version upgrade
- **Type 3 (Bug/Support)**: Reactive fix

Assign:
- **Demo date**: Target demo date
- **Dev/QA pairing**: Assigned developer and QA engineer

---

### Step 2: Load Context (3-layer KB + codebase)

**2a. Read `{AGENTS_MD}`** — project conventions, constraints, patterns to follow/avoid.

**2b. Match feature keywords against KB (3-layer)**

The agent's session-start KB scan provides general awareness. Now match the **feature description keywords** against those indexes to find docs specific to this plan. The goal is to reference relevant docs in task descriptions and identify knowledge gaps — not to deep-read everything.

1. **Layer 1 — Framework**: Match feature keywords against `{FRAMEWORK_KB_DIR}/_index.md` triggers (e.g. architecture, networking, persistence). Note matching doc names — include them as "Relevant KB" in task descriptions so implementers load the right docs.
2. **Layer 2 — Container**: Check `{CONTAINER_KB_DIR}/README.md` for product domain docs, shared API contracts, or engineering standards related to the feature. If a domain concept is central to the feature and not documented, note it as a KB gap for the plan.
3. **Layer 3 — Project**: Check `{KB_DIR}/README.md` for project-specific architecture, integration, or feature docs that provide context for how to implement.

If a KB doc is directly relevant to the feature (e.g. planning a payments feature and `product/domain/subscription-model.md` exists), read that doc — it may contain domain rules that affect task design.

**2c. Explore codebase**

1. Explore relevant source files in `{PROJECT_CODE_ROOT}`:
   - Identify which layer/module owns the change
   - Find existing patterns similar to what needs to be built
   - Look for data model changes that require migrations
2. Identify files that will need to be created or modified
3. Identify existing tests for the affected area
4. Note any KB/docs that should be updated (include as task deliverables)

---

### Step 3: Generate Plan Slug

Create a kebab-case slug (2-5 words) from the feature description.

Examples: `offline-reading`, `pdf-viewer`, `user-auth`, `api-refactor`

---

### Step 4: Break Into Tasks

Design tasks with these principles:

1. **Clear file ownership** — Each task owns specific files. No two concurrent tasks modify the same file
2. **Layer separation** — Tasks in different layers can often run in parallel
3. **Dependency ordering** — Data model changes before business logic; business logic before UI
4. **Right-sized** — Completable in a single agent session (~1-20 file changes)
5. **Zero-padded IDs** — `task-01`, `task-02`, etc.
6. **Test and docs ownership** — Every implementation task owns its test coverage and any KB updates

For each task, populate from `references/plan-task.md`:
- **Task ID**: `task-{NN}`
- **Task Path**: unique slug (e.g., `task-01-data-model`, `task-02-api-layer`)
- **JIRA**: Link to JIRA ticket or "N/A"
- **File Ownership**: specific files this task creates or modifies
- **Do NOT Modify**: cross-reference files owned by sibling tasks in the same phase
- **Testing**: what to test and how
- **Documentation / KB Updates**: existing docs to update or note for document-solution

Generate **kill criteria** — 2-3 conditions that would stop work entirely.

**Sizing validation**: Warn if plan exceeds 15 tasks or 4 phases — suggest splitting into multiple plans.

---

### Step 5: Group Into Phases

Typical phase structure:
- **Phase 1**: Foundation (data models, API layer, schema migrations)
- **Phase 2**: Business logic (services, state management, ViewModels)
- **Phase 3**: UI (screens, components, navigation)
- **Phase 4**: Polish + tests (if separated from implementation)

Adjust based on the project's architecture.

---

### Step 5a: Resolve Master Plan (If Linked)

Skip this step if `--master-plan` was not specified and the user did not indicate a master plan link.
Set the `Master Plan` field in `overview.md` to "None" if skipped.

In this framework, a **master plan** is a plan at the container root (`{PLANS_DIR}/{master-plan-slug}/`).
Local plans (the ones being created here) live in a tech/project subfolder (`{PLANS_DIR}/{PROJECT_DIR}/{slug}/`).
Everything is local within `{PLANS_DIR}` — no external repository required.

1. Check that `{PLANS_DIR}/{master-plan-slug}/overview.md` exists.
   If not found locally, check the hub:
   a. Resolve `HUB_DIR` from `.ai-framework.config` (`HUB_DIR` key).
      If `HUB_DIR` is not set or the directory is unreachable, stop and ask the user to verify the slug or create the master plan first.
   b. Scan all state directories in the hub for `plans/{state}/{master-plan-slug}/plan.md`
      (states: `active`, `completed`, `deprioritized`, `cancelled`).
   c. If found in `active` or `completed`: read `plan.md` for scope and expectations to carry into the local plan.
   d. If found in `deprioritized` or `cancelled`: warn the user that the master plan is **{state}** and ask for explicit confirmation before proceeding.
   e. If not found in any state directory (and not found locally): stop and ask the user to verify the slug or create the master plan first.
2. Read `{PLANS_DIR}/{master-plan-slug}/overview.md` (or the hub `plan.md`) for overall scope, success criteria, and any
   quality/documentation expectations to carry into the local plan.
3. Carry any master-plan expectations into the generated `overview.md` and `task.md` files.
4. Set the `Master Plan` field in the local plan's `overview.md` to the master plan slug.

---

### Step 6: Create Directory Structure

Write plan files to `{PLANS_DIR}/{slug}/`:

```
# Flat (≤5 tasks):
{PLANS_DIR}/{slug}/
  overview.md
  task-XX-{slug}/
    task.md
    status.md

# Phased (>5 tasks):
{PLANS_DIR}/{slug}/
  overview.md
  phase-N/
    task-XX-{slug}/
      task.md
      status.md
```

Reference templates in `.ai-framework/skills/create-plan/references/` if they exist:
- `references/plan-overview.md`
- `references/plan-task.md`
- `references/plan-task-status.md`

---

### Step 7: Update Plans Index

Add to `{PLANS_DIR}/README.md` Active Plans table:

```markdown
| [{Plan Title}]({slug}/overview.md) | {Objective} | 0/{N} | not-started | {YYYY-MM-DD} |
```

If `{PLANS_DIR}/README.md` does not exist, create it with a basic index structure first.

If `PLANS_MODE=repo`: remind the user to commit the plans repo separately.

---

### Step 8: Report Summary

Present the plan:
- Plan name, slug, type classification
- Demo date and Dev/QA assignments
- Master plan linkage (if any)
- Number of phases and tasks
- Kill criteria
- Dependency graph (which tasks can run in parallel)
- Planned test / KB coverage
- Branch convention: `plan/{slug}/{task-path}`
- Suggest next step: `/execute-plan {slug}` or `/execute-task {slug}/{task-path}`
- If linked to a master plan, note that the master plan at `{PLANS_DIR}/{master-plan-slug}/` will be updated on each task completion via Step 9 of `execute-task`

## Output

- `{PLANS_DIR}/{slug}/` directory with overview.md and task directories
- Updated `{PLANS_DIR}/README.md` index
- Summary of plan structure presented to user
