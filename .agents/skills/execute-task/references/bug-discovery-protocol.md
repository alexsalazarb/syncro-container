# Bug Discovery Protocol

When a bug is discovered during implementation, triage using this table:

| Situation | Action |
|-----------|--------|
| Small bug in code you're changing | Fix inline; add regression test; document as adaptation |
| Larger bug that blocks your task | Add defect task (below); mark this task `blocked` |
| Pre-existing, doesn't block | Log in plan's Defects table; continue |
| Unrelated bug | Do NOT add to plan; continue |

## Adding a Defect Task (when it blocks)

1. Scan all task directories (including `phase-*/`) to find the highest `task-{NN}` number across the entire plan
2. Determine placement:
   - Flat plan: place in plan root as `{PLANS_DIR}/{plan-slug}/{defect-task-path}`
   - Phased plan: place in the same phase as the blocking task
3. Create directory with `task.md` (fields: Plan, Task ID, Task Path, Depends On, Blocks, JIRA, Severity, Found By, Found During, Bug Description, Root Cause, File Ownership, Implementation Steps, Testing, Completion Criteria) and `status.md` (`not-started`)
4. Add to `overview.md` Task Summary table and Defects table
5. Update current task's `status.md`: add blocker, set `blocked`
6. Commit: `plan({slug}): add defect task {defect-task-path}`
7. Inform user and stop. Fix the defect task first, then resume this task.

**Note**: Agent-discovered bugs use this protocol. Human-reported bugs use `/add-defect {plan-slug}` instead.
