# Task: Regression tests for TabController initialization flow

**Plan**: Fix Appointments TabController LateInitializationError
**Phase**: 2
**Task ID**: task-02
**Task Path**: task-02-regression-tests
**Depends On**: task-01-guard-tab-controller
**JIRA**: N/A

## Objective

Write unit tests that verify `AppointmentsListCubit` behaves safely when `refresh()` and
`disposeView()` are called before `setTabController()`, and that calling `setTabController()`
followed by `refresh()` / `disposeView()` works as expected.

## Context

Task 01 makes `tabController` nullable and adds `_tabControllerReady` guards to `refresh()` and
`disposeView()`. These paths have no test coverage. The test file should live alongside existing
cubit tests in the appointments home test directory.

Existing test patterns:
- Tests under `syncro-flutter/test/features/appointments/appointments_home/` use
  `mockito`-generated mocks and `flutter_test`.
- Use `bloc_test` (`blocTest`) for cubit stream assertions.
- Use `flutter_test` `TestWidgetsFlutterBinding` for any widget-related scaffolding.

See the flutter-architecture cubit conventions:
`syncro-flutter/.cursor/rules/flutter-architecture.mdc` — states are sealed Equatable subclasses,
cubits use `safeEmit()`.

## Before You Start

- [ ] Switch to base branch and verify task-01 is complete: check `task-01-guard-tab-controller/status.md`
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read `appointments_list_cubit.dart` post-task-01 changes before writing tests
- [ ] Scan `test/features/appointments/appointments_home/` for mock generation patterns to follow
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/test/features/appointments/appointments_home/application/appointments_list_cubit_test.dart` | create | New test file for cubit initialization and guard paths |

### Do NOT Modify

- Any source files under `lib/` — task-01 owns all source changes
- Any existing test files

## Implementation Steps

### Step 1: Scaffold the test file

Create `test/features/appointments/appointments_home/application/appointments_list_cubit_test.dart`.

Follow the mock/import patterns used in adjacent test files (e.g.,
`get_appointments_usecase_test.dart`). You will need a mock for `GetAppointmentsUseCase` and a
real (or mock) `AppointmentsFilterCubit`.

If mocks need to be regenerated, run:
```bash
cd syncro-flutter && flutter pub run build_runner build --delete-conflicting-outputs
```

### Step 2: Write test group — unguarded access safety

```dart
group('tabController not yet set', () {
  test('refresh() returns early without throwing', () async {
    // arrange: cubit created, setTabController never called
    // act: call refresh()
    // assert: no exception, no state change
  });

  test('disposeView() returns early without throwing', () async {
    // arrange: cubit created, setTabController never called
    // act: call disposeView()
    // assert: no exception, _tabControllerReady remains false
  });
});
```

### Step 3: Write test group — normal flow

```dart
group('after setTabController()', () {
  test('setTabController sets _tabControllerReady to true', () {
    // arrange: cubit + real TabController (requires TickerProvider)
    // act: setTabController(controller)
    // assert: tabController is non-null, _tabControllerReady is true
  });

  test('disposeView() clears tabController and resets ready flag', () {
    // arrange: cubit with tab controller set
    // act: disposeView()
    // assert: tabController is null, _tabControllerReady is false
  });

  test('refresh() delegates to mineController when tab index is 0', () {
    // arrange: cubit with tabController at index 0
    // act: refresh()
    // assert: mineController receives refresh signal
  });

  test('refresh() delegates to allController when tab index is 1', () {
    // arrange: cubit with tabController at index 1
    // act: refresh()
    // assert: allController receives refresh signal
  });
});
```

### Step 4: Run all appointment tests

```bash
cd syncro-flutter && flutter test test/features/appointments/
```

All tests must pass. Fix any failures before marking task complete.

## Testing

- [ ] `refresh()` called before `setTabController()` — no exception thrown
- [ ] `disposeView()` called before `setTabController()` — no exception thrown
- [ ] `setTabController()` + `refresh()` delegates to the correct paging controller by tab index
- [ ] `disposeView()` nulls the tabController and resets `_tabControllerReady`
- [ ] All existing appointment tests still pass
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required for this task

## Completion Criteria

- [ ] New test file created at the path above
- [ ] All new tests pass
- [ ] No existing tests broken
- [ ] `flutter analyze` passes
- [ ] Changes committed to `plan/fix-appointments-tab-controller/task-02-regression-tests` branch
- [ ] Status updated in `status.md`
