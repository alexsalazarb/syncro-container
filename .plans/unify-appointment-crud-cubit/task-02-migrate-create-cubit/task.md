# task-02: Migrate AppointmentCreateCubit to Use Base Shared Methods

**Task ID**: task-02  
**Plan**: unify-appointment-crud-cubit  
**Depends On**: task-01  
**Branch**: `plan/unify-appointment-crud-cubit/task-02-migrate-create-cubit`

---

## Objective

Update `AppointmentCreateCubit` to implement the abstract methods defined in task-01 and remove the now-duplicate `_determineLocation()`, `selectAppointmentType()`, `sendCustomerEmailChanged()` and `determineAppointmentLocationUseCase` field.

Also add a `ticketIdOf` helper getter to `AppointmentCreateLoaded` so the base cubit can read the ticket ID without knowing about `TicketData`.

---

## File Ownership

**Modifies**:
- `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart`
- `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_state.dart`

**Do NOT Modify** (owned by task-01):
- `appointment_crud_base_cubit.dart`

**Do NOT Modify** (owned by task-03):
- `appointment_edit_cubit.dart`
- `appointment_edit_state.dart`

---

## Steps

### 1. Add `crudTicketId` getter to `AppointmentCreateLoaded` (state file)

In `appointment_create_state.dart`, add a getter to `AppointmentCreateLoaded`:

```dart
/// Ticket ID for use by CrudAppointmentBaseCubit — delegates to ticketData.
int? get crudTicketId => ticketData?.id;
```

### 2. Pass `determineAppointmentLocationUseCase` to super constructor

In `AppointmentCreateCubit`, update the `super(...)` call:

```dart
AppointmentCreateCubit({
  // ... same params
  required this.determineAppointmentLocationUseCase,
  // ...
}) : super(
  AppointmentCreateLoading(),
  determineAppointmentLocationUseCase: determineAppointmentLocationUseCase,
) {
  init(data);
}
```

**Remove** the `final DetermineAppointmentLocationUseCase determineAppointmentLocationUseCase;` field — it now lives in the base.

### 3. Implement the 10 abstract methods

Add these implementations to `AppointmentCreateCubit`:

```dart
// ── Abstract method implementations ─────────────────────────────────────────

@override
AppointmentCreateLoaded? get loadedStateOrNull =>
    state is AppointmentCreateLoaded ? state as AppointmentCreateLoaded : null;

@override
AppointmentCreateState buildPopulateLocationState(String location) =>
    AppointmentCreatePopulateLocation(location);

@override
AppointmentCreateState withDeterminingLocation(
  AppointmentCreateState loaded, {required bool value}) =>
    (loaded as AppointmentCreateLoaded).copyWith(isDeterminingLocation: value);

@override
AppointmentCreateState withLocationFromApi(
  AppointmentCreateState loaded, String location) =>
    (loaded as AppointmentCreateLoaded).copyWith(locationFromApi: location);

@override
AppointmentCreateState withSelectedType(
  AppointmentCreateState loaded, AppointmentTypeOption type) =>
    (loaded as AppointmentCreateLoaded).copyWith(selectedAppointmentType: type);

@override
AppointmentCreateState withClearedTypeAndLocation(
  AppointmentCreateState loaded) =>
    (loaded as AppointmentCreateLoaded).copyWith(
      clearSelectedAppointmentType: true,
      clearLocationFromApi: true,
    );

@override
AppointmentCreateState withSendCustomerEmail(
  AppointmentCreateState loaded, bool value) =>
    (loaded as AppointmentCreateLoaded).copyWith(sendCustomerEmail: value);

@override
AppointmentTypeOption? selectedTypeOf(AppointmentCreateState loaded) =>
    (loaded as AppointmentCreateLoaded).selectedAppointmentType;

@override
Customer? customerOf(AppointmentCreateState loaded) =>
    (loaded as AppointmentCreateLoaded).customer;

@override
int? ticketIdOf(AppointmentCreateState loaded) =>
    (loaded as AppointmentCreateLoaded).crudTicketId;
```

### 4. Remove duplicate methods

Delete from `AppointmentCreateCubit`:
- `sendCustomerEmailChanged(bool? value)` — now in base
- `selectAppointmentType(AppointmentTypeOption? type)` — now in base
- `_determineLocation()` — now in base

### 5. Update `customerChanged` and `ticketChanged` to use `_determineLocation` from base

These methods already call `await _determineLocation()` — they will now call the base implementation. No change needed to the call site; just verify they still compile.

### 6. Run format and analyze

```bash
fvm dart format syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart
fvm dart format syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_state.dart
fvm flutter analyze syncro-flutter/lib/features/appointments/appointment_create/
fvm flutter test test/features/appointments/appointment_create/
```

---

## Testing

Run existing tests for `appointment_create/` — no new tests needed here since no new behavior is introduced. Tests for the shared `_determineLocation` logic are written in task-03.

---

## Acceptance Criteria

- [ ] `AppointmentCreateCubit` no longer defines `_determineLocation()`, `selectAppointmentType()`, or `sendCustomerEmailChanged()`
- [ ] `determineAppointmentLocationUseCase` field removed from `AppointmentCreateCubit`
- [ ] All 10 abstract methods implemented
- [ ] `crudTicketId` getter added to `AppointmentCreateLoaded`
- [ ] `fvm flutter analyze` passes for `appointment_create/`
- [ ] All existing `appointment_create/` tests pass
