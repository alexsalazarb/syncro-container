# Plan: Move Priority Resolution to Domain Model

**Status**: complete
**Created**: 2026-04-13
**Last Updated**: 2026-04-13
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: feature/SE-11864

## Objective

Move `Priority.from` / `Priority.labelFrom` resolution out of widget `build()` methods into computed getters on the `Ticket` and `TicketDetails` domain models, eliminating per-frame recomputation that contributes to list item flicker.

## Scope

### In Scope
- Add `resolvedPriority` and `priorityLabel` computed getters to `Ticket`
- `TicketDetails` inherits the getters automatically (extends `Ticket`)
- Update `TicketItem` widget to use the new getters
- Update `DetailTabPageSection.priority` case to use the new getter
- Unit tests for the new getters on `Ticket`

### Out of Scope
- `getStatusToShow()` logic — this is a settings-based lookup (not a pure string transform), correctly lives in the cubit/presentation layer
- `Priority.labelFrom` usage in `ticket_detail_tab_page.dart` line 357 — operates on `priorities` list from settings, not on a `Ticket` field
- Flicker caused by cubit double-emit (loading → loaded) — separate concern, not addressed here

## Kill Criteria

- If the API changes `priority` from a string to a structured object, the getter approach would need redesign
- If `Priority.from` needs `BuildContext` or async data in the future, it can't be a domain getter

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Domain | task-01 | None | Add computed getters to Ticket model |
| 2 | Presentation | task-02 | task-01 | Update widgets to use new getters |
| 3 | Testing | task-03 | task-01 | Unit tests for the new getters |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| task-01-domain-priority-getters | Add priority getters to Ticket model | 1 | complete | -- |
| task-02-update-presentation-callers | Update widgets to use model getters | 2 | complete | task-01 |
| task-03-unit-tests | Unit tests for Ticket priority getters | 3 | complete | task-01 |

## Branch Convention

Pattern: `plan/ticket-priority-domain-getters/{task-path}`

Examples:
- `plan/ticket-priority-domain-getters/task-01-domain-priority-getters`
- `plan/ticket-priority-domain-getters/task-02-update-presentation-callers`

Base branch: confirmed by developer when running `execute-plan`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart` | Domain model — receives the new getters |
| `syncro-flutter/lib/features/ticket/ticket_details/domain/ticket_details.dart` | Extends Ticket — inherits getters automatically |
| `syncro-flutter/lib/core/enums/priority.dart` | Priority enum with `from()` and `labelFrom()` |
| `syncro-flutter/lib/features/ticket/ticket_home/presentation/widget/ticket_item.dart` | Widget calling Priority.from in build() |
| `syncro-flutter/lib/features/ticket/ticket_details/domain/detail_tab_section.dart` | DetailTab calling Priority.labelFrom |
| `syncro-flutter/test/features/ticket/ticket_home/domain/ticket_test.dart` | Existing tests for Ticket model |

## Risks

- Minimal risk — pure refactor moving computation location without changing behavior
- `TicketDetails` extends `Ticket`, so getters are inherited automatically with no extra work

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] `ticket.resolvedPriority` returns same value as `Priority.from(ticket.priority)`
- [ ] `ticket.priorityLabel` returns same value as `Priority.labelFrom(ticket.priority)`
- [ ] `TicketItem.build()` no longer calls `Priority.from` / `Priority.labelFrom` directly
- [ ] `DetailTabPageSection.priority` uses `details.priorityLabel` instead of `Priority.labelFrom`
- [ ] All existing tests pass
- [ ] New unit tests cover null, empty, known, and unknown priority values

## References

- **JIRA Epic**: N/A
- **Related Plans**: None
