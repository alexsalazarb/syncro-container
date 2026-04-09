# Plan: [BUG TITLE]

<!-- Replace [BUG TITLE] with a clear description of the bug being fixed -->

**Status**: not-started | in-progress | complete
**Created**: [DATE]
**Last Updated**: [DATE]
**Type**: Bug Fix (Type 3)
**Severity**: P0 | P1 | P2 | P3
<!-- P0: Data loss/corruption, system down — hotfix track
     P1: Feature broken, many users, no workaround — expedited
     P2: Feature degraded, workaround exists — standard flow
     P3: Cosmetic, single user, edge case — lower priority -->
**Ticket**: [TICKET-KEY or "N/A"]
**Assigned Dev**: [NAME]
**Assigned QA**: [NAME or "unassigned"]
**Trigger**: [ticket, monitoring alert, or internal discovery]
**Blast Radius**: [N users affected, or "unknown"]
**Master Plan**: {master-slug or "None"}
<!-- Set by create-bug-plan when a cross-project bug is detected.
     Points to {PLANS_DIR}/{master-slug}/ at the container root. -->

## Bug Summary

[1-2 sentence description of the bug — what's broken and who's affected]

## Root Cause

[2-5 sentence explanation of the actual root cause. Include the specific file, line, and what the code does wrong. Reference investigation.md for full details.]

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| [System 1] | [What it does] | [How the bug/fix affects it] |

## Scope

### In Scope
- [Fix item 1]
- [Regression test]
- [Consumer verification, if applicable]

### Out of Scope
- [Item 1] — {reason}

## Kill Criteria

- Fix introduces worse behavior than the original bug
- Root cause is disproved by new evidence
- [Specific condition related to this bug]

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-[slug] | [Fix title] | not-started | — |
| task-02-regression-tests | Add regression tests | not-started | — |

## Branch Convention

Task branches: `plan/{slug}/{task-path}`
Merge target: `main`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| [path] | **Primary fix** — [why] |

## Success Criteria

- [ ] Bug no longer reproducible with original repro steps
- [ ] Regression test fails before fix, passes after
- [ ] All existing tests pass
- [ ] All consumers handle the changed behavior gracefully
- [ ] Required KB / documentation updates complete or explicitly marked not needed

## Defects

<!-- Bugs discovered during plan execution. Added by execute-task's Bug Discovery Protocol (Step 5a). -->

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

<!-- Verified by execute-task when the last task completes. Do not remove items. -->
- [ ] All tasks complete or adapted
- [ ] Bug no longer reproducible with original repro steps
- [ ] Regression test: red before fix, green after (verified)
- [ ] All existing tests pass
- [ ] investigation.md root cause matches the actual fix (no drift)
- [ ] All consumers handle the changed behavior (if applicable)
- [ ] KB/documentation updated or explicitly marked not needed
- [ ] Ticket transitioned (or transition noted for manual action)
- [ ] Staging verification complete

## Revert Plan

**Revert trigger**: [Specific condition — e.g., "error rate increases by >10%"]
**Revert steps**: [How to roll back]
**Rollback owner**: [Who monitors and executes if needed]

## References

- **Ticket**: [link or "N/A"]
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: [links or "None"]
