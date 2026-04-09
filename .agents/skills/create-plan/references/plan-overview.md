# Plan: [PLAN TITLE]

<!-- Replace [PLAN TITLE] with a clear, outcome-oriented name -->

**Status**: not-started | in-progress | complete
**Created**: [DATE]
**Last Updated**: [DATE]
**Estimated Demo Date**: [YYYY-MM-DD or "TBD"]
**Assigned Dev**: [NAME or "unassigned"]
**Assigned QA**: [NAME or "unassigned"]
**Master Plan**: {slug or "None"}
**Integration Branch**: plan/{slug} — populated by `execute-plan` at start; "N/A" when `PR_INTEGRATION=false`
**Base Branch**: main — developer-confirmed when running `execute-plan`; may be overridden at runtime

<!-- Status values: not-started | in-progress | complete -->

## Objective

[1-2 sentence description of what this plan accomplishes]

## Scope

### In Scope
- [Item 1]
- [Item 2]

### Out of Scope

- [Item 1] — {reason excluded}
- [Item 2] — {reason excluded}

## Kill Criteria

- {e.g., "If the third-party SDK drops support for our minimum OS version"}
- {e.g., "If the API changes make the feature technically infeasible within the sprint"}
- {e.g., "If performance testing shows the approach won't meet the latency requirement"}

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Foundation | task-01, task-02 | None | Data models and API layer |
| 2 | Logic | task-03, task-04 | Phase 1 | Business logic and services |
| 3 | UI | task-05 | Phase 2 | Screens and user-facing components |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| phase-1/task-01-data-model | [Title] | 1 | not-started | — |
| phase-1/task-02-api-layer | [Title] | 1 | not-started | — |
| phase-2/task-03-business-logic | [Title] | 2 | not-started | phase-1/task-01-data-model |
| phase-3/task-04-ui | [Title] | 3 | not-started | phase-2/task-03-business-logic |

## Branch Convention

Pattern: `plan/[PLAN-SLUG]/{task-path}`

Examples:
- `plan/[PLAN-SLUG]/phase-1/task-01-data-model`
- `plan/[PLAN-SLUG]/phase-2/task-03-business-logic`

Base branch: confirmed by developer when running `execute-plan` (see `**Base Branch**` field above)

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `[path/to/models]` | Data models |
| `[path/to/services]` | Business logic |
| `[path/to/api]` | Network / API layer |
| `[path/to/screens]` | UI components |

## Risks

- Risk 1 — Mitigation
- Risk 2 — Mitigation

## Defects

<!-- Bugs discovered during plan execution. Added by execute-task's Bug Discovery Protocol (Step 5a). -->

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] All tests pass
- [ ] KB/documentation updates complete or explicitly marked not needed

## References

- **JIRA Epic**: {JIRA-EPIC-KEY or "N/A"}
- **Related Plans**: {links to related active plans or "None"}
