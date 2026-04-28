# task-03: Migrate AppointmentEditCubit to Use Base Shared Methods + Add Tests

**Task ID**: task-03  
**Plan**: unify-appointment-crud-cubit  
**Depends On**: task-01  
**Branch**: `plan/unify-appointment-crud-cubit/task-03-migrate-edit-cubit`

---

## Objective

Update `AppointmentEditCubit` to implement the abstract methods defined in task-01, remove duplicate logic, and add cubit-level tests covering the shared `_determineLocation()` and `selectAppointmentType()` behavior (which now lives in the base).

---

## File Ownership

**Modifies**:
- `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart`
- `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_state.dart`
- `syncro-flutter/test/features/appointments/appointment_edit/appointment_edit_cubit_test.dart` (new file)
- `syncro-flutter/test/features/appointments/appointment_edit/appointment_edit_cubit_test.mocks.dart` (generated)

**Do NOT Modify** (owned by task-01):
- `appointment_crud_base_cubit.dart`

**Do NOT Modify** (owned by task-02):
- `appointment_create_cubit.dart`
- `appointment_create_state.dart`

---

## Steps

### 1. Pass `determineAppointmentLocationUseCase` to super constructor

In `AppointmentEditCubit`, update the super call:

```dart
AppointmentEditCubit({
  // ... same params
}) : super(
  AppointmentEditLoading(),
  determineAppointmentLocationUseCase: determineAppointmentLocationUseCase,
) {
  getAppointment();
}
```

**Remove** `final DetermineAppointmentLocationUseCase determineAppointmentLocationUseCase;` field.

### 2. Implement the 10 abstract methods

```dart
// ── Abstract method implementations ─────────────────────────────────────────

@override
AppointmentEditLoaded? get loadedStateOrNull =>
    state is AppointmentEditLoaded ? state as AppointmentEditLoaded : null;

@override
AppointmentEditState buildPopulateLocationState(String location) =>
    AppointmentEditPopulateLocation(location);

@override
AppointmentEditState withDeterminingLocation(
  AppointmentEditState loaded, {required bool value}) =>
    (loaded as AppointmentEditLoaded).copyWith(isDeterminingLocation: value);

@override
AppointmentEditState withLocationFromApi(
  AppointmentEditState loaded, String location) =>
    (loaded as AppointmentEditLoaded).copyWith(locationFromApi: location);

@override
AppointmentEditState withSelectedType(
  AppointmentEditState loaded, AppointmentTypeOption type) =>
    (loaded as AppointmentEditLoaded).copyWith(selectedAppointmentType: type);

@override
AppointmentEditState withClearedTypeAndLocation(
  AppointmentEditState loaded) =>
    (loaded as AppointmentEditLoaded).copyWith(
      clearSelectedAppointmentType: true,
      clearLocationFromApi: true,
    );

@override
AppointmentEditState withSendCustomerEmail(
  AppointmentEditState loaded, bool value) =>
    (loaded as AppointmentEditLoaded).copyWith(sendCustomerEmail: value);

@override
AppointmentTypeOption? selectedTypeOf(AppointmentEditState loaded) =>
    (loaded as AppointmentEditLoaded).selectedAppointmentType;

@override
Customer? customerOf(AppointmentEditState loaded) =>
    (loaded as AppointmentEditLoaded).customer;

@override
int? ticketIdOf(AppointmentEditState loaded) =>
    (loaded as AppointmentEditLoaded).ticketId;
```

### 3. Remove duplicate methods

Delete from `AppointmentEditCubit`:
- `sendCustomerEmailChanged(bool? value)` — now in base
- `selectAppointmentType(AppointmentTypeOption? type)` — now in base
- `_determineLocation()` — now in base

### 4. Run format and analyze

```bash
fvm dart format syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart
fvm flutter analyze syncro-flutter/lib/features/appointments/
fvm flutter test
```

### 5. Generate mocks and write cubit test

```bash
fvm flutter pub run build_runner build --delete-conflicting-outputs
```

Create `test/features/appointments/appointment_edit/appointment_edit_cubit_test.dart` with:

**Test group 1 — `selectAppointmentType()`**

- When called with a non-null type:
  - emits state with `selectedAppointmentType` set
  - then calls `_determineLocation()` (mock the use case to return a location)
  - emits `isDeterminingLocation = true`
  - emits `AppointmentEditPopulateLocation(location)`
  - emits final state with `isDeterminingLocation = false` and `locationFromApi` set

- When called with `null`:
  - emits state with `selectedAppointmentType` cleared
  - does NOT call `determineAppointmentLocationUseCase`

**Test group 2 — stale-check in `_determineLocation()`**

- When the appointment type changes while `_determineLocation()` is awaiting:
  - the populate location state is NOT emitted

**Test group 3 — `sendCustomerEmailChanged()`**

- When called with `true` → emits state with `sendCustomerEmail = true`
- When called with `false` → emits state with `sendCustomerEmail = false`

Use `bloc_test` package (`blocTest`) and `mockito` for the use cases. Follow the existing pattern in `test/features/appointments/appointments_home/application/appointments_list_cubit_test.dart`.

---

## Testing

```bash
fvm flutter test test/features/appointments/appointment_edit/appointment_edit_cubit_test.dart
fvm flutter test  # full suite — no regressions
```

---

## Acceptance Criteria

- [ ] `AppointmentEditCubit` no longer defines `_determineLocation()`, `selectAppointmentType()`, or `sendCustomerEmailChanged()`
- [ ] `determineAppointmentLocationUseCase` field removed from `AppointmentEditCubit`
- [ ] All 10 abstract methods implemented
- [ ] `appointment_edit_cubit_test.dart` created with the 5 test cases described above
- [ ] `fvm flutter analyze` passes (zero errors/warnings)
- [ ] `fvm flutter test` passes (full suite)
