# Plan: syncro-codebase-improvements

**Created**: April 2026
**Status**: Active
**Project**: syncro-flutter
**Branch Convention**: `plan/syncro-codebase-improvements/task-0X`

## Objective

Address critical and major code quality issues found during the April 2026 audit:
1. Token refresh is not implemented (critical — users can have silent auth failures)
2. DI double-registration causes inconsistent repository instances
3. `WorksheetRepository` interface registration bug

## Scope

- **In scope**: Token refresh implementation, DI cleanup, WorksheetRepository fix
- **Out of scope**: Localization, navigation timing delays, inter-cubit dependency refactor (lower priority)

## Findings Source

`docs/kb-projects/syncro-flutter/ai-patterns/known-issues.md`

## Dependency DAG

```
task-01-implement-token-refresh
  └─ No dependencies (self-contained)

task-02-fix-di-double-registration
  └─ No dependencies (self-contained)
  
task-03-fix-worksheet-repository-type (can run parallel with task-01 and task-02)
  └─ No dependencies
```

Tasks are independent and can be executed in any order or in parallel.

## Task Summary

| Task | Slug | Priority | Status |
|------|------|----------|--------|
| 01 | implement-token-refresh | 🔴 Critical | complete |
| 02 | fix-di-double-registration | 🟠 Major | adapted (pre-empted by SE-11671) |
| 03 | fix-worksheet-repository-type | 🟠 Major | not-started |
| 04 | rename-edit-custom-fields-dir | 🟡 Minor | not-started |
