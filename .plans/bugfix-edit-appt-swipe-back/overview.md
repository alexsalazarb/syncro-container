# Plan: Edit Appointment — Swipe-to-Back Does Not Navigate After Dialog

**Status**: complete
**Created**: 2026-05-04
**Last Updated**: 2026-05-04
**Type**: Bug Fix (Type 3)
**Severity**: P2
**Ticket**: N/A
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Trigger**: Internal discovery
**Blast Radius**: All users of Edit Appointment screen (iOS swipe-to-back gesture)
**Master Plan**: None

## Bug Summary

On the Edit Appointment screen, performing the iOS swipe-to-back gesture shows the "Leave Screen" dialog correctly, but tapping "Leave Screen" to confirm does not navigate back — the user stays on the Edit Appointment screen. Create Appointment works correctly.

## Root Cause

`AppointmentEditPage` wraps its content in two nested `AppScaffold` widgets, both with `showOnBackDialog: true`. Each `AppScaffold` generates a `PopScope(canPop: false)` widget. When the swipe-to-back gesture fires, Flutter dispatches `onPopInvokedWithResult` to **both** `PopScope` instances simultaneously. After the inner dialog accepts and calls `routeCubit.goBack()`, the outer `PopScope(canPop: false)` intercepts the resulting pop attempt — blocking navigation or triggering a second, unexpected dialog. `AppointmentCreatePage` works because it has a single `AppScaffold`. See [investigation.md](investigation.md) for full details.

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| `appointment_edit_page.dart` | Edit Appointment screen entry point | Primary fix — remove duplicate PopScope |
| `app_scaffold.dart` | Wraps children in `PopScope(canPop: false)` when `showOnBackDialog: true` | Not modified; behavior is correct in isolation |

## Scope

### In Scope
- Remove `showOnBackDialog: true` and `backNavigationConfig` from the outer `AppScaffold` in `AppointmentEditPage`
- Regression widget test: swipe-to-back on Edit Appointment → accept dialog → navigates back

### Out of Scope
- Refactoring `AppointmentEditPage` to match `AppointmentCreatePage` structure — desirable but separate concern
- Auditing other pages for the same double-AppScaffold pattern — separate task

## Kill Criteria

- Fix breaks the existing Create Appointment swipe-back flow
- Fix breaks the Delete button flow in Edit Appointment
- Root cause is disproved by new evidence

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-fix-nested-popscope | Remove duplicate showOnBackDialog from outer AppScaffold | complete | — |
| task-02-regression-tests | Add regression widget test for swipe-back on Edit Appointment | complete | task-01-fix-nested-popscope |

## Branch Convention

Task branches: `plan/bugfix-edit-appt-swipe-back/{task-path}`
Merge target: `main`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart` | **Primary fix** — outer AppScaffold has spurious showOnBackDialog: true |
| `syncro-flutter/lib/core/global_widgets/app_scaffold.dart` | Reference — PopScope logic lives here |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart` | Reference — correct single-AppScaffold pattern |

## Success Criteria

- [ ] Swipe-to-back on Edit Appointment → dialog shows → "Leave Screen" → navigates back
- [ ] Regression test fails before fix, passes after
- [ ] All existing appointment tests pass
- [ ] Create Appointment swipe-back unaffected
- [ ] KB / documentation updates complete or explicitly marked not needed

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

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

**Revert trigger**: Swipe-back on any appointment screen stops working, or new nav regression detected in smoke testing.
**Revert steps**: `git revert` the fix commit; no data migration needed.
**Rollback owner**: Developer on call.

## References

- **Ticket**: N/A
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: None
- **Introducing commit**: `adfc91d9 SCLP-690: Mobile App: Show "Leave Screen" confirmation dialog when navigating back from creation/edit screens`
