# Plan: Fix Appointments TabController LateInitializationError

**Status**: not-started
**Created**: 2026-04-09
**Last Updated**: 2026-04-10
**Estimated Demo Date**: TBD
**Assigned Dev**: Alex Salazar
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: main

## Objective

Fix the `LateInitializationError: Field 'tabController' has not been initialized` crash that occurs the first time the Appointments section is opened. The crash is caused by unguarded accesses to a `late TabController` field in a globally-scoped cubit whose controller is only set when the view's `initState()` runs.

## Root Cause

`AppointmentsListCubit` is a **global cubit** (registered in `app_providers.dart` at app startup). Its `late TabController tabController` field is only populated when `AppointmentsView.initState()` calls `setTabController()`. Several methods access this field without first checking the `_tabControllerReady` flag:

| Location | Unguarded access |
|---|---|
| `AppointmentsListCubit.refresh()` | `tabController.index` — no guard |
| `AppointmentsListCubit.disposeView()` | `tabController.removeListener` / `tabController.dispose()` — no guard |
| `AppointmentsView.build()` (lines 59, 66, 71) | `listCubit.tabController` — runtime-safe only if initState ran |

`_handleFilterChange` and `tabListener` already use `_tabControllerReady` correctly; the pattern just was not applied consistently.

## Scope

### In Scope
- Make `tabController` nullable (`TabController? tabController`) in `AppointmentsListCubit`
- Add null/readiness guards to `refresh()`, `disposeView()`, and all other cubit methods that touch the controller
- Update `AppointmentsView.build()` to handle the nullable controller safely (null-check where accessed outside of the guaranteed `initState` path)
- Regression tests for the unguarded paths

### Out of Scope
- Architectural refactor to move `TabController` ownership entirely into the `StatefulWidget` State — valid long-term improvement but out of scope for this targeted fix
- Changes to `AppointmentsFilterCubit`, paging controllers, or use cases

## Kill Criteria

- If a required architectural change (moving `TabController` out of the cubit) is mandated by the team before this fix can ship, stop and create a new plan for the refactor
- If the reproduction scenario is found to require a fundamentally different fix, stop and revise the plan

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Fix | task-01-guard-tab-controller | None | Make tabController nullable and guard all accesses |
| 2 | Tests | task-02-regression-tests | task-01-guard-tab-controller | Write cubit unit tests covering unguarded paths |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| task-01-guard-tab-controller | Guard tabController accesses in cubit and view | 1 | not-started | — |
| task-02-regression-tests | Regression tests for TabController initialization flow | 2 | not-started | task-01-guard-tab-controller |

## Branch Convention

Pattern: `plan/fix-appointments-tab-controller/{task-path}`

Examples:
- `plan/fix-appointments-tab-controller/task-01-guard-tab-controller`
- `plan/fix-appointments-tab-controller/task-02-regression-tests`

## Key Files

| File/Directory | Relevance |
|---|---|
| `syncro-flutter/lib/features/appointments/appointments_home/application/appointments_list_cubit.dart` | Cubit — primary fix target |
| `syncro-flutter/lib/features/appointments/appointments_home/presentation/appointments_view.dart` | View — contains unguarded `listCubit.tabController` calls in `build()` and BlocListener |
| `syncro-flutter/lib/app/dependency/app_providers.dart` | Registers the cubit as a global provider — confirms global lifecycle |
| `syncro-flutter/test/features/appointments/appointments_home/` | Existing test directory for this feature |

## Risks

- Making `tabController` nullable could require type adjustments in widgets that consume it (`AppointmentsTabBar`, `TabBarView`) — handled in task-01 with a null-safe access pattern in the view

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|---|---|---|---|---|

## Success Criteria

- [ ] Opening the Appointments section for the first time does not crash
- [ ] `refresh()` called before `setTabController()` returns early without crashing
- [ ] `disposeView()` called before `setTabController()` returns early without crashing
- [ ] All existing tests pass
- [ ] New regression tests cover the unguarded access paths
- [ ] `flutter analyze` passes with no new warnings

## References

- **JIRA Ticket**: [SE-11855](https://repairtechsolutions.atlassian.net/browse/SE-11855)
- **Related Plans**: None
