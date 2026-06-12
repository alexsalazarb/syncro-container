# Task: Add Regression Tests for Appointment Time Change Flows

**Plan**: SE-12739-appointment-time-revert
**Task ID**: task-03
**Task Path**: task-03-regression-tests
**Depends On**: task-01-fix-modal-time-capture, task-02-fix-edit-cubit-refresh
**Ticket**: SE-12739

## Objective

Write regression tests that fail before the task-01 + task-02 fixes are applied and pass after, covering: (1) `AppointmentEditCubit` refresh methods do NOT re-emit `DateRadioButton.time`; (2) time change is correctly saved in the Create cubit; (3) time change is correctly saved in the Edit cubit.

## Context

See [investigation.md](../investigation.md) for the root cause. Tests should cover the exact failure modes:
- Edit cubit's `startRadioButtonTypeRefresh` / `endRadioButtonTypeRefresh` must terminate without re-emitting `DateRadioButton.time`
- `changeStartTime` / `changeEndTime` update cubit state correctly (these likely already pass — verify or add if missing)

Existing test files to check first:
- `syncro-flutter/test/features/appointments/appointment_create/application/appointment_create_cubit_test.dart`
- `syncro-flutter/test/features/appointments/appointment_edit/application/appointment_edit_cubit_test.dart`

If these files don't exist yet, create them.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-01 and task-02 are complete
- [ ] Read [investigation.md](../investigation.md) to understand what to assert
- [ ] Check existing test files for appointments — don't duplicate passing tests
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/test/features/appointments/appointment_edit/application/appointment_edit_cubit_test.dart` | create or modify | Primary regression tests |
| `syncro-flutter/test/features/appointments/appointment_create/application/appointment_create_cubit_test.dart` | create or modify | Verify create cubit is unaffected |

### Do NOT Modify

- Any production code files — tests only
- Any files in `syncro-flutter/lib/` — owned by task-01 and task-02

## Implementation Steps

### Step 1: Set up test file(s)

Check if `appointment_edit_cubit_test.dart` exists. If not, create it with standard boilerplate:
- Import `flutter_test`, `bloc_test`, `mockito`
- Mock all use cases injected into the cubit
- Create a helper that builds `AppointmentEditLoaded` with `startRadioButtonType: DateRadioButton.time` and a known `startTime`

### Step 2: Test — `startRadioButtonTypeRefresh` does NOT re-emit `DateRadioButton.time`

```dart
blocTest<AppointmentEditCubit, AppointmentEditState>(
  'startRadioButtonTypeRefresh: when startRadioButtonType is time, '
  'emits removeStartButtonType then terminates without re-emitting time',
  build: () => buildCubitWithState(
    startRadioButtonType: DateRadioButton.time,
    startTime: const TimeOfDay(hour: 10, minute: 0),
  ),
  act: (cubit) async {
    await cubit.startRadioButtonTypeRefresh();
  },
  expect: () => [
    isA<AppointmentEditLoaded>().having(
      (s) => s.startRadioButtonType,
      'startRadioButtonType',
      isNull,  // ← only this emit, no re-emit of DateRadioButton.time
    ),
  ],
);
```

Do the same for `endRadioButtonTypeRefresh`.

### Step 3: Test — `changeStartTime` saves the new time

```dart
blocTest<AppointmentEditCubit, AppointmentEditState>(
  'changeStartTime: updates startTime in state',
  build: () => buildCubitWithState(
    startTime: const TimeOfDay(hour: 9, minute: 0),
    endTime: const TimeOfDay(hour: 10, minute: 0),
  ),
  act: (cubit) => cubit.changeStartTime(const TimeOfDay(hour: 11, minute: 30)),
  expect: () => [
    isA<AppointmentEditLoaded>().having(
      (s) => s.startTime,
      'startTime',
      const TimeOfDay(hour: 11, minute: 30),
    ),
  ],
);
```

Do the same for `changeEndTime`.

### Step 4: Test — `changeStartTime` in Create cubit saves the new time

Mirror Step 3 for `AppointmentCreateCubit` to confirm it's unaffected and remains correct.

### Step 5: Run all tests

```bash
cd syncro-flutter
fvm flutter test test/features/appointments/ --no-pub
```

All tests must pass.

## Testing

- [ ] New regression tests FAIL on unpatched code (before task-01 + task-02 fixes) — this should be verifiable by reverting task-01/task-02 temporarily if needed
- [ ] New regression tests PASS after fixes
- [ ] All existing appointment tests still pass
- [ ] `fvm flutter analyze` passes
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required for this task

## Completion Criteria

- [ ] `startRadioButtonTypeRefresh` regression test: verifies no re-emit of `DateRadioButton.time`
- [ ] `endRadioButtonTypeRefresh` regression test: same
- [ ] `changeStartTime` test: verifies time is saved
- [ ] `changeEndTime` test: same
- [ ] All new tests pass: `fvm flutter test test/features/appointments/`
- [ ] Changes committed to `plan/SE-12739-appointment-time-revert/task-03-regression-tests` branch
- [ ] Status updated in `status.md`
