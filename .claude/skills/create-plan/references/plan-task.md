# Task: [TASK TITLE]

**Plan**: [Plan name]
**Phase**: [Phase number]
**Task ID**: task-{NN}
**Task Path**: [task-path] (e.g., `task-01-data-model` or `phase-1/task-01-data-model`)
**Depends On**: [{task-path}] or "None"
**JIRA**: {JIRA-TICKET-KEY} or "N/A"

## Objective

[1-2 sentence description of what this task accomplishes]

## Context

<!-- If JIRA or docs contradict this task file, the live sources win. Adapt and document
     the deviation in status.md. -->
[Background information needed to execute this task. Reference relevant files, existing patterns, or architecture decisions. The agent should be able to execute this task by reading just this file plus the referenced documents.]

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] If `JIRA` is not "N/A", read ticket {JIRA-TICKET-KEY}: check description AND comments for latest requirements
- [ ] Verify every prerequisite task in `Depends On` is complete (check `status.md` for each)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read referenced architecture docs or KB entries
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| [path/to/file] | create / modify | [Brief note] |

### Do NOT Modify

- `{path/to/file}` — owned by {task-path}

## Implementation Steps

### Step 1: [Action]
[Detailed instructions]

### Step 2: [Action]
[Detailed instructions]

## Testing

- [ ] [Test requirement 1 — specific behavior or unit to verify]
- [ ] [Test requirement 2]
- [ ] Existing tests still pass
- [ ] Linter / static analysis passes (run the project's standard lint command)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Update existing docs / KB files affected by this task, or explicitly note "No KB/doc updates required"
- [ ] If no KB doc exists and this task introduces a reusable or non-obvious pattern, run `document-solution`
- [ ] If KB files change, run `check-kb-index`

## Completion Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] All tests pass
- [ ] No regressions in existing functionality
- [ ] Documentation / KB updates completed or explicitly marked not needed
- [ ] Changes committed to `plan/[plan-slug]/[task-path]` branch
- [ ] Status updated in `status.md`
