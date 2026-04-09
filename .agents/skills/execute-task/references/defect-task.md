# Task: [DEFECT TITLE]

**Plan**: [Plan name]
**Task ID**: task-[NN]
**Task Path**: task-[NN]-defect-[slug]
**Depends On**: "None"
**Blocks**: [task-path of the blocked task]
**JIRA**: [JIRA-KEY] or "N/A"
**Discovered During**: [task-path that found this bug]
**Discovery Context**: [1-2 sentences: what were you doing when you found this]

## Bug Description

[What's broken, where, and why it blocks the parent task]

## Root Cause

[Brief root cause — the discovering agent already knows this]

**File**: [path] (lines [N-M])
**What it does**: [current behavior]
**What it should do**: [expected behavior]

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| [path] | modify | [Brief note] |

## Implementation Steps

### Step 1: Fix
[Specific fix instructions]

### Step 2: Regression Test
[Test that fails before fix, passes after]

## Completion Criteria

- [ ] Bug fixed
- [ ] Regression test added (fails before, passes after)
- [ ] Existing tests pass
- [ ] Changes committed to `plan/[slug]/[task-path]` branch
- [ ] Status updated in `status.md`
- [ ] Blocked task unblocked (remove blocker from its status.md)
