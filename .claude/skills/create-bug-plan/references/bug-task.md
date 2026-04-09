# Task: [TASK TITLE]

**Plan**: [Bug plan name]
**Task ID**: task-[NN]
**Task Path**: task-[NN]-[slug]
**Depends On**: [task-path] or "None"
**Ticket**: [TICKET-KEY] or "N/A"

## Objective

[1-2 sentence description of what this task accomplishes]

## Context

<!-- For bug fix tasks, reference the investigation.md findings.
     The executing agent should understand the full bug context from this file
     plus investigation.md. -->
[Reference investigation.md findings. Include specific file paths and line numbers.]

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] If `Ticket` is not "N/A", read the ticket: check description AND comments for latest requirements
- [ ] Verify every prerequisite task in `Depends On` is complete
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read [investigation.md](../investigation.md) for full root cause context
- [ ] Mark this task `in-progress` in `status.md` before proceeding

<!-- If ticket or docs contradict this task file, the live sources win. Adapt and document
     the deviation in status.md. -->

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| [path] | modify | [Brief note] |

### Do NOT Modify

- `[path/to/file]` — owned by [task-path]

## Implementation Steps

### Step 1: [Action]
[Detailed instructions]

### Step 2: [Action]
[Detailed instructions]

## Testing

<!-- For the regression test task, this IS the main deliverable.
     For the fix task, verify existing tests still pass. -->
- [ ] [Test requirement 1]
- [ ] Existing tests still pass
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Update existing docs / KB files affected by this task, or explicitly note "No KB/doc updates required"
- [ ] If this task introduces a reusable or non-obvious pattern, run `document-solution`
- [ ] If KB files change, run `check-kb-index`

## Completion Criteria

- [ ] [Criterion 1]
- [ ] Documentation / KB updates completed or explicitly marked not needed
- [ ] Changes committed to `plan/[slug]/[task-path]` branch
- [ ] Status updated in `status.md`
