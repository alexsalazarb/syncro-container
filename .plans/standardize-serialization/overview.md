# Plan: Standardize Domain Model Serialization (fromJson / toJson)

**Status**: not-started
**Created**: 2026-04-09
**Last Updated**: 2026-04-09
**Estimated Demo Date**: TBD — technical cleanup, no user-facing demo
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: main

## Objective

Migrate every domain model in syncro-flutter from the inconsistent mix of `fromMap`/`fromJson`/`toMap`/`toJson` naming to the single canonical convention: `fromJson(Map<String, dynamic>)` and `toJson()`. Remove the `fromJson(String source)` aliases that parse raw JSON strings — those callers should use `jsonDecode()` inline instead.

## Scope

### In Scope
- Rename all `fromMap(Map<String, dynamic>)` factory constructors to `fromJson`
- Rename all `toMap()` methods to `toJson()`
- Remove `fromJson(String source)` factory aliases; update every call site to use `jsonDecode()` inline
- Update all corresponding test files and infrastructure deserializer/repository call sites
- Collapse classes that had both `fromMap` and `fromJson(String source)` into a single canonical `fromJson(Map<String, dynamic>)`

### Out of Scope
- `json_serializable` or `freezed` code-generation adoption — convention change only, no tooling switch
- Changes to the Cursor rule (`flutter-architecture.mdc`) — already updated in the session prior to this plan
- Any non-domain, non-infrastructure Dart files (UI widgets, cubits, pages) — they do not call serialization methods directly

## Kill Criteria

- If a class is found to have two genuinely distinct `fromMap` / `fromJson` constructors with different semantics (not just aliases), stop and document the exception before proceeding
- If `flutter analyze` reports more than 10 new errors that are not straightforward rename fallout, escalate to the team before continuing

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Core Shared Models | task-01 | None | Customer, Contact, TinyContact, Settings, Locale + their infra and test call sites |
| 2 | Feature Domain Models | task-02, task-03, task-04 | Phase 1 | Ticket-home, Ticket-details, Appointments & misc — can run in parallel |
| 3 | Infrastructure Sweep | task-05 | Phase 2 | Catch all remaining `fromMap`/`toMap`/`fromJson(String)` occurrences across the codebase |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| phase-1/task-01-core-shared-models | Core shared domain models | 1 | not-started | — |
| phase-2/task-02-ticket-home-models | Ticket-home domain models | 2 | not-started | phase-1/task-01 |
| phase-2/task-03-ticket-details-models | Ticket-details domain models | 2 | not-started | phase-1/task-01 |
| phase-2/task-04-appointments-misc-models | Appointments & misc domain models | 2 | not-started | phase-1/task-01 |
| phase-3/task-05-infrastructure-sweep | Infrastructure & final sweep | 3 | not-started | phase-2/* |

## Branch Convention

Pattern: `plan/standardize-serialization/{task-path}`

Examples:
- `plan/standardize-serialization/phase-1/task-01-core-shared-models`
- `plan/standardize-serialization/phase-2/task-02-ticket-home-models`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/features/customers/domain/customer.dart` | Customer, Contact, Address, CustomerOverview — all have fromMap/fromJson(String) |
| `syncro-flutter/lib/features/settings/locale/domain/settings.dart` | Settings, TicketSettings, TicketType — all use fromMap |
| `syncro-flutter/lib/features/ticket/ticket_details/` | Largest set of models to migrate |
| `syncro-flutter/lib/features/*/infrastructure/` | All deserializer files that call .fromMap() |
| `syncro-flutter/.cursor/rules/flutter-architecture.mdc` | Rule already updated — canonical reference for the convention |

## Risks

- Test regressions — Mitigation: each task runs `flutter analyze` and the relevant test suite before marking complete
- Missed call sites — Mitigation: task-05 does a final codebase-wide grep as a safety net
- `fromJson(String)` removal breaking serialization roundtrips — Mitigation: every removal includes updating the single caller to use `jsonDecode(source)` inline

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] Zero occurrences of `fromMap(` in any `domain/` or `infrastructure/` Dart file
- [ ] Zero occurrences of `.toMap()` in any `domain/` or `infrastructure/` Dart file
- [ ] Zero occurrences of `fromJson(String` (raw-string overload) in any domain file
- [ ] All existing tests pass (`flutter test`)
- [ ] `flutter analyze` reports no new errors or warnings

## References

- **JIRA Epic**: N/A
- **Related Plans**: None
- **Convention source**: `syncro-flutter/.cursor/rules/flutter-architecture.mdc` — Domain Model Serialization section
