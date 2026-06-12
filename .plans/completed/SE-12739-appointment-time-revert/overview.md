# Plan: Appointment Time Changes Revert After Pressing OK

**Status**: complete
**Created**: 2026-06-11
**Last Updated**: 2026-06-12
**Type**: Bug Fix (Type 3)
**Severity**: P1
**Ticket**: SE-12739
**Assigned Dev**: Alex Salazar
**Assigned QA**: unassigned
**Trigger**: Customer report — Adam Gravett (adamgravett.syncromsp.com) + internal QA verification
**Blast Radius**: All users of Create and Edit Appointment screens (iOS and Android)
**Master Plan**: None

## Bug Summary

When creating or editing an appointment, changing the start or end time in the time picker modal and pressing "OK" has no effect — the time reverts to its original value. The date field works correctly. Reproducible on iOS and Android, affects both Create and Edit flows.

## Root Cause

Regression introduced by commit **5207f5ee** (SCLP-809, ~Feb 2026), which replaced the inline `CustomTimePicker` widget (with "Save Time" button) with a modal `showCustomTimePickerModal` (using Flutter's `showTimePicker`). Three confirmed defects were introduced (plus one unrelated hotfix):

1. **Primary (both flows)** — In `custom_datetime_picker.dart`, the `.then()` callback called `onModeChange(null)` BEFORE `onTimeChanged`. This meant the cubit's radio button state was reset before the new time was saved. The reset triggered `startRadioButtonTypeRefresh()` (Edit cubit), which after 500 ms re-emitted `DateRadioButton.time` with `widget.time` still holding the original value — reopening the picker with the old time and discarding the selection.

2. **Secondary (Edit flow only)** — `AppointmentEditCubit.startRadioButtonTypeRefresh` and `endRadioButtonTypeRefresh` re-emitted `DateRadioButton.time` after a 500 ms delay, causing `didUpdateWidget` in `CustomDateTimePicker` to open a second modal with `initTime = widget.time` (the original time, since the save hadn't landed yet). The Create cubit's equivalent methods were already correct.

3. **Bonus (date change path)** — `TimezoneService.convertToTimezone` applied the timezone offset twice for non-UTC datetimes: `.toUtc()` + `tz.TZDateTime.from(utcDateTime, location)`. When the user changed the appointment **date**, the start/end time shifted by the device's UTC offset.

See [investigation.md](investigation.md) for full analysis.

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| `custom_datetime_picker/custom_datetime_picker.dart` | Shows modal via `didUpdateWidget` + `addPostFrameCallback` | **Primary fix** — swap callback order: `onTimeChanged` before `onModeChange(null)` |
| `custom_datetime_picker/custom_time_picker.dart` | `showCustomTimePickerModal` — wraps `showTimePicker` | Move `MediaQuery` wrapper into builder; remove unnecessary `async` |
| `custom_datetime_picker/time_picker.dart` | Custom copy of Flutter time picker — `CustomTimePickerDialog._handleOk` | Fix missing `form.save()` call (dead code correctness fix) |
| `appointment_edit/application/appointment_edit_cubit.dart` | Edit cubit — `startRadioButtonTypeRefresh` / `endRadioButtonTypeRefresh` | Remove spurious re-emit of `DateRadioButton.time` |
| `core/services/timezone_service.dart` | Timezone conversion for appointment datetimes | Fix double timezone conversion for non-UTC datetimes (hotfix) |
| `appointment_create/application/appointment_create_cubit.dart` | Create cubit — same methods | **No change needed** — already correct |

## Scope

### In Scope
- Swap callback order in `_CustomDateTimePickerState.didUpdateWidget` `.then()`: call `onTimeChanged` before `onModeChange(null)`
- Move `MediaQuery` wrapper inside `showCustomTimePickerModal` builder; remove `async` from function
- Fix `CustomTimePickerDialog._handleOk` to call `form.save()` before reading `_selectedTime.value`
- Remove re-emit of `DateRadioButton.time` from EDIT cubit's refresh methods
- Fix double timezone conversion in `TimezoneService.convertToTimezone` for non-UTC datetimes
- Regression tests for both Create and Edit time change flows

### Out of Scope
- Refactoring `AppointmentDateTimeSection` or unifying Create/Edit cubits — separate concern
- Switching from `showTimePicker` to a fully custom dialog — unnecessary if `PopScope` fix resolves the primary cause
- Auditing other places that use `showCustomTimePickerModal` — only used in appointment flows

## Kill Criteria

- Fix causes the time picker to be undismissible in a way that blocks the user
- Root cause is disproved (time reverts even with `PopScope` + cubit fix)
- Fix breaks existing Create Appointment or Edit Appointment tests

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-fix-modal-time-capture | Fix time modal capture and dead-code form.save() | complete | — |
| task-02-fix-edit-cubit-refresh | Remove re-emit in Edit cubit refresh methods | complete | — |
| task-03-regression-tests | Add regression tests for time change flows | complete | task-01, task-02 |

## Branch Convention

Task branches: `plan/SE-12739-appointment-time-revert/{task-path}`
Merge target: `main`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_datetime_picker.dart` | **Primary fix** — swap callback order in `.then()` |
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_time_picker.dart` | `MediaQuery` wrapper placement + remove `async` |
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/time_picker.dart` | Fix `_handleOk` missing `form.save()` (dead code) |
| `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` | Fix `startRadioButtonTypeRefresh` / `endRadioButtonTypeRefresh` |
| `syncro-flutter/lib/core/services/timezone_service.dart` | Hotfix — double timezone conversion on date change |

## Success Criteria

- [ ] Time changes save correctly in Create Appointment on iOS and Android
- [ ] Time changes save correctly in Edit Appointment on iOS and Android
- [ ] Back gesture on the time picker modal does NOT silently dismiss it
- [ ] Regression test fails before the fix, passes after
- [ ] All existing appointment tests pass
- [ ] KB / documentation updated or explicitly marked not needed

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

- [ ] All tasks complete or adapted
- [ ] Bug no longer reproducible with original repro steps (iOS + Android)
- [ ] Regression test: red before fix, green after (verified)
- [ ] All existing tests pass
- [ ] investigation.md root cause matches the actual fix (no drift)
- [ ] KB/documentation updated or explicitly marked not needed
- [ ] Ticket SE-12739 transitioned to "Ready for QA"
- [ ] Staging verification complete

## Revert Plan

**Revert trigger**: Time picker becomes undismissible or blocks appointment save on either platform
**Revert steps**: Revert the three files changed in task-01 + task-02; cherry-pick is cleanest
**Rollback owner**: Alex Salazar

## References

- **Ticket**: https://syncrotech.atlassian.net/browse/SE-12739
- **Investigation**: [investigation.md](investigation.md)
- **Regression commit**: `5207f5ee` (SCLP-809)
- **Related Plans**: None
