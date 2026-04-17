# Development Plans

Git-native development plans for this repository. Each plan is a self-contained directory with an overview, tasks, and status tracking.

## Active Plans

| Plan | Objective | Project | Tasks | Status | Target Demo |
|------|-----------|---------|-------|--------|-------------|
| [fix-production-crashes-v140](fix-production-crashes-v140/overview.md) | Fix 4 production crashes from Crashlytics v1.4.0 (FATAL + null-safety) | syncro-flutter | 0/4 | not-started | TBD |
| [syncro-codebase-improvements](syncro-codebase-improvements/overview.md) | Fix critical/major issues found in April 2026 audit | syncro-flutter | 4 | not-started | — |
| [search-delegate-pagination](search-delegate-pagination/overview.md) | Paginated search with pull-to-refresh in CustomSearchDelegate | syncro-flutter | 0/4 | not-started | TBD |
| [standardize-serialization](standardize-serialization/overview.md) | Standardize all domain model serialization to fromJson/toJson | syncro-flutter | 0/5 | not-started | TBD |

## Backlog

| Plan | Objective | Project | Priority | Added |
|------|-----------|---------|----------|-------|
<!-- Plans captured but not yet started -->

## Completed Plans

| Plan | Objective | Project | Tasks | Completed |
|------|-----------|---------|-------|-----------|
| [fix-appointments-tab-controller](completed/fix-appointments-tab-controller/overview.md) | Fix LateInitializationError on first Appointments open | syncro-flutter | 2/2 | 2026-04-10 |
| [fix-gorouter-cubit-extras](completed/fix-gorouter-cubit-extras/overview.md) | Eliminate GoRouter cubit-as-extra serialization warnings via TicketCubitRegistry | syncro-flutter | 4/4 | 2026-04-13 |
| [ticket-priority-domain-getters](completed/ticket-priority-domain-getters/overview.md) | Move Priority resolution from widget build() to Ticket model getters | syncro-flutter | 3/3 | 2026-04-13 |
| [appointment-location-type](appointment-location-type/overview.md) | Send appointment_location_type on create/update via dedicated /appointment_types endpoint | syncro-flutter | 4/4 | 2026-04-14 |

---

## How It Works

### Lifecycle

1. **Create** — Run `/create-plan` to generate a plan from a feature description
2. **Define Contracts** — Run `/manage-contracts` to define shared interfaces between tasks before parallel execution begins
3. **Execute** — Run `/execute-task {plan}/{task}` to work on a specific task, or `/execute-plan {plan}` to run all remaining tasks
4. **Track** — Each task has a `status.md` that tracks progress (not-started -> in-progress -> complete)
5. **Complete** — When all tasks finish, the plan moves to the Completed table above

### Directory Structure

#### Flat Layout (Default)

```text
.plans/
  README.md                          # This file (index of all plans)
  {plan-slug}/                       # One directory per plan
    overview.md                      # Scope, phases, dependency DAG, branch convention
    contracts/                       # Shared interfaces between parallel tasks
      {contract-name}.md             # Interface definition (API shape, data format, etc.)
    task-01-{slug}/                  # One directory per task
      task.md                        # Objective, steps, file ownership, criteria
      status.md                      # Current state, timestamps, agent, PR link
    task-02-{slug}/
      task.md
      status.md
```

#### Phased Layout (For Larger Plans)

When a plan has many tasks, grouping by phase improves navigability:

```text
.plans/
  README.md
  {plan-slug}/
    overview.md
    contracts/
      api-response-format.md         # Shared interface agreed upon before work starts
      auth-token-schema.md
    phase-1/
      task-01-{slug}/
        task.md
        status.md
      task-02-{slug}/
        task.md
        status.md
    phase-2/
      task-03-{slug}/
        task.md
        status.md
      task-04-{slug}/
        task.md
        status.md
    phase-3/
      task-05-{slug}/
        task.md
        status.md
```

The `contracts/` directory holds interface definitions that parallel tasks agree on before execution begins. This prevents integration failures when tasks within the same phase are worked on by different agents concurrently.

### Branch Convention

Plans use the branch pattern: `plan/{plan-slug}/{task-id}`

Example: `plan/user-auth/task-01`

### Status Values

| Status | Meaning |
|--------|---------|
| `not-started` | Task has not been picked up |
| `in-progress` | Task is actively being worked on |
| `complete` | Task is finished and verified |
| `blocked` | Task cannot proceed (see blockers in status.md) |
| `adapted` | Task was modified from original spec during execution |

### Parallel Execution

Tasks within the same phase (no cross-dependencies) can be executed in parallel using agent teams. Each task defines a **file ownership** list to prevent merge conflicts between concurrent agents. Shared **contracts** define agreed-upon interfaces so parallel tasks integrate cleanly.

### Integration with Skills

- `/create-plan` — Creates a new plan with tasks, phases, and dependencies
- `/execute-task` — Executes a single task (checks deps, creates branch, updates status)
- `/execute-plan` — Executes all remaining tasks with parallel agent support
- `/manage-contracts` — Defines or updates shared interface contracts between parallel tasks
- `/create-master-plan` — Creates a master plan in the hub INDEX (container layouts)
- `/transition-plan` — Moves plans between lifecycle states (backlog → planned → active → complete)
- `/archive-plan` — Archives a completed plan
- `pre-commit-check` — Runs automatically before task commits
