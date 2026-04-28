# Plan: Unify Appointment CRUD Cubit Logic

**Status**: complete  
**Type**: Type 2 — Technical (Refactor)  
**Project**: syncro-flutter  
**Branch Convention**: `plan/unify-appointment-crud-cubit/{task-path}`  
**Master Plan**: None  
**Target Demo**: TBD  
**Dev**: Unassigned  
**QA**: Unassigned  
**Last Updated**: 2026-04-28

---

## Objective

Move `selectAppointmentType()`, `_determineLocation()`, and `sendCustomerEmailChanged()` from both `AppointmentCreateCubit` and `AppointmentEditCubit` into their shared base class `CrudAppointmentBaseCubit<T>`.

The two cubits are ~95% identical in these three methods. The only difference is which state types they reference (`AppointmentCreateLoaded` vs `AppointmentEditLoaded`, `AppointmentCreatePopulateLocation` vs `AppointmentEditPopulateLocation`). The base cubit delegates state manipulation to child cubits via abstract methods.

**Scope**: Cubit layer only. Views, state initialization, and state classes are NOT modified (except adding one helper getter to `AppointmentCreateLoaded`).

---

## Success Criteria

1. `_determineLocation()` exists only in `CrudAppointmentBaseCubit` — removed from both child cubits
2. `selectAppointmentType()` exists only in `CrudAppointmentBaseCubit` — removed from both child cubits
3. `sendCustomerEmailChanged()` exists only in `CrudAppointmentBaseCubit` — removed from both child cubits
4. `determineAppointmentLocationUseCase` field moved from child cubits to base
5. `fvm flutter analyze` passes with zero errors
6. `fvm flutter test` passes (no regressions)

---

## Kill Criteria

1. The abstract method interface requires modifying views or the presentation layer → stop and redesign
2. The state types cannot satisfy the abstract interface without breaking existing tests → stop
3. The refactor requires changing more than 5 files outside the cubit layer → stop and reconsider scope

---

## Architecture Decision

`CrudAppointmentBaseCubit<T>` gains abstract methods for state manipulation. Each child cubit implements these one-liners. The base cubit owns the algorithm; the children own the types.

```
CrudAppointmentBaseCubit<T>
  ├── abstract: loadedStateOrNull → T?
  ├── abstract: buildPopulateLocationState(String) → T
  ├── abstract: withDeterminingLocation(T, bool) → T
  ├── abstract: withLocationFromApi(T, String) → T
  ├── abstract: withSelectedType(T, AppointmentTypeOption) → T
  ├── abstract: withClearedTypeAndLocation(T) → T
  ├── abstract: withSendCustomerEmail(T, bool) → T
  ├── abstract: selectedTypeOf(T) → AppointmentTypeOption?
  ├── abstract: customerOf(T) → Customer?
  ├── abstract: ticketIdOf(T) → int?
  ├── concrete: _determineLocation()
  ├── concrete: selectAppointmentType(AppointmentTypeOption?)
  └── concrete: sendCustomerEmailChanged(bool?)
```

---

## Phase / Task Summary

| Task | Path | Depends On | Status |
|------|------|------------|--------|
| task-01 | task-01-extend-base-cubit | — | not-started |
| task-02 | task-02-migrate-create-cubit | task-01 | not-started |
| task-03 | task-03-migrate-edit-cubit | task-01 | not-started |

Tasks 02 and 03 can run in **parallel** after task-01 completes — they touch different files.

---

## Dependency Graph

```
task-01 (extend base)
   ├── task-02 (migrate create)   ─┐
   └── task-03 (migrate edit)    ─┘  (parallel)
```
