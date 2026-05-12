# Task 02 — Regression Tests for Active Session Detection

## Objective

Write regression tests for `LatestTimeClockCubit` that prove:
1. When the list contains an open session AND a more recently modified historical record, `isClockInActive` returns `false` (user is clocked in).
2. When no open session exists, `isClockInActive` returns `true` (show Clock In).
3. When the list is empty, `isClockInActive` returns `true` (safe default).

These tests must **fail before task-01** is applied and **pass after**.

## Context

- **Cubit under test**: `LatestTimeClockCubit` (`lib/features/time_clock/application/latest_time_clock_cubit.dart`)
- **Use case to mock**: `GetLatestTimeClockUsecase`
- **Test file (new)**: `test/features/time_clock/application/latest_time_clock_cubit_test.dart`
- **Mock file (generated)**: `test/features/time_clock/application/latest_time_clock_cubit_test.mocks.dart`
- **Pattern**: Follow `test/features/chat/application/new_chat_cubit_test.dart` — `@GenerateMocks`, `bloc_test`, `GetIt.instance` teardown in `tearDown`

## Implementation Steps

1. Create `test/features/time_clock/application/` directory.

2. Create `latest_time_clock_cubit_test.dart` with `@GenerateMocks([GetLatestTimeClockUsecase, AddTimeClockUsecase])`.

3. Write the following test cases:

   **Scenario A — open session wins over recently edited historical record**
   - `GetLatestTimeClockUsecase` returns a `TimeClockResponse` where `timeClock.isClockedIn == true` (has `inAt`, no `outAt`)
   - Assert: `cubit.isClockInActive == false`
   - Assert: `cubit.isClockOutActive == true`
   - Assert: emitted state is `LatestTimeClockLoaded` with `timeClock.outAt == null`

   **Scenario B — no open session → shows Clock In**
   - `GetLatestTimeClockUsecase` returns a `TimeClockResponse` with `timeClock.outAt != null` (a closed record)
   - Assert: `cubit.isClockInActive == true`
   - Assert: `cubit.isClockOutActive == false`

   **Scenario C — null timeClock → shows Clock In (safe default)**
   - `GetLatestTimeClockUsecase` returns `TimeClockResponse(timeClock: null)`
   - Assert: `cubit.isClockInActive == true`

4. Run code generation:
   ```bash
   cd syncro-flutter && fvm flutter pub run build_runner build --delete-conflicting-outputs
   ```

5. Run tests:
   ```bash
   cd syncro-flutter && fvm flutter test test/features/time_clock/application/latest_time_clock_cubit_test.dart
   ```

## File Ownership

**MUST create:**
- `test/features/time_clock/application/latest_time_clock_cubit_test.dart`
- `test/features/time_clock/application/latest_time_clock_cubit_test.mocks.dart` (generated)

**MUST NOT modify:**
- Any production source file
- Any other test file

## Dependencies

- task-01 must be complete (tests are written against the fixed behaviour)

## Acceptance Criteria

- [ ] Three test scenarios written and passing
- [ ] Scenario A fails against the pre-fix code (validates regression coverage)
- [ ] `fvm flutter test test/features/time_clock/` passes clean
- [ ] No `flutter analyze` warnings introduced
