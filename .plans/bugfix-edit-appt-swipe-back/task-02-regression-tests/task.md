# Task: Add Regression Test — Edit Appointment Swipe-Back

**Plan**: bugfix-edit-appt-swipe-back
**Task ID**: task-02
**Task Path**: task-02-regression-tests
**Depends On**: task-01-fix-nested-popscope
**Ticket**: N/A

## Objective

Add a widget test that verifies swipe-to-back on `AppointmentEditPage` shows the Leave Screen dialog and, upon acceptance, navigates back. The test must fail on the unfixed code (two nested PopScopes) and pass after task-01.

## Context

See [investigation.md](../investigation.md) for full root cause details.

The bug was caused by a double `PopScope(canPop: false)` in `AppointmentEditPage`. The regression test must:
1. Mount `AppointmentEditPage` with the minimum required providers
2. Simulate a back gesture via `WidgetTester.pageBack()` or by invoking the system back button
3. Verify the "Leave Screen" dialog appears
4. Tap the "Leave Screen" / accept button
5. Verify navigation back occurred (check that the page is no longer on screen, or that `goBack` was called)

Check the existing test suite for `AppointmentCreatePage` back navigation tests — if any exist, mirror their pattern for `AppointmentEditPage`.

## Before You Start

- [ ] Switch to `plan/bugfix-edit-appt-swipe-back/task-01-fix-nested-popscope` branch and verify task-01 is complete
- [ ] Read [investigation.md](../investigation.md)
- [ ] Check existing appointment test files for mock/provider setup patterns:
  - `syncro-flutter/test/features/appointments/`
- [ ] Check AGENTS.md for test conventions: `syncro-flutter/AGENTS.md`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/test/features/appointments/appointment_edit/presentation/appointment_edit_page_test.dart` | create | New regression test file |

### Do NOT Modify

- Any production source file — task-01 owns those
- Existing test files unless adding a helper used by this test

## Implementation Steps

### Step 1: Locate existing appointment test patterns

```bash
find syncro-flutter/test -name "*appointment*" -type f
```

Read any matching files to understand the mock setup pattern (provider injection, RouteCubit mocking, BlocProvider wrapping).

### Step 2: Check AGENTS.md for test conventions

Read `syncro-flutter/AGENTS.md` for:
- Test framework (flutter_test + bloc_test + mockito)
- Mock generation command
- Naming conventions

### Step 3: Write the regression test

Create `syncro-flutter/test/features/appointments/appointment_edit/presentation/appointment_edit_page_test.dart`.

The test must:
1. Provide a mock `RouteCubit` (verify `goBack()` is called on accept)
2. Provide mock cubits for `AppointmentEditCubit` and `AppointmentTypesCubit`
3. Pump `AppointmentEditPage`
4. Simulate back gesture: `await tester.pageBack()` or send `SystemNavigator.pop()` equivalent
5. Verify "Leave Screen" dialog appears (find by text `AppStrings.leaveDialogTitle` or `AppStrings.yesLeave`)
6. Tap the accept button
7. Verify `mockRouteCubit.goBack()` was called once

### Step 4: Generate mocks if needed

If new mocks are required:

```bash
cd syncro-flutter
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

### Step 5: Run the test

```bash
cd syncro-flutter
fvm flutter test test/features/appointments/appointment_edit/presentation/appointment_edit_page_test.dart --reporter expanded
```

All assertions must pass.

## Testing

- [ ] Regression test passes after task-01 fix
- [ ] (Verification step) Confirm test fails when `showOnBackDialog: true` is added back to the outer AppScaffold — demonstrates the test is a real regression guard
- [ ] Full appointment test suite passes: `fvm flutter test test/features/appointments/`
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

No KB/doc updates required.

## Completion Criteria

- [ ] Regression test file created and passes
- [ ] Test verifies: dialog shown + accept + `goBack()` called
- [ ] Full appointment test suite green
- [ ] Changes committed to `plan/bugfix-edit-appt-swipe-back/task-02-regression-tests` branch
- [ ] Status updated to `complete` in `status.md`
