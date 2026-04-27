# Task 02: Create Cubit Integration

**Plan**: appointment-determine-location
**Phase**: 2 — State
**Status**: not-started
**Depends on**: phase-1/task-01-domain-networking

## Objective

Integrate `DetermineAppointmentLocationUseCase` into `AppointmentCreateCubit` so that the `location` field is auto-filled via API whenever appointment type, customer, or ticket changes.

## Steps

1. **Inject use case** via constructor:
   ```dart
   AppointmentCreateCubit({
     ...existing params...,
     required this.determineAppointmentLocationUseCase,
   })
   ```

2. **Add `isDeterminingLocation: bool` to `AppointmentCreateLoaded`** (defaults to `false`).

3. **Create private helper `_determineLocation()`**:
   - Read `selectedAppointmentType`, `customer?.id`, `ticketData?.id` from current state
   - If `selectedAppointmentType == null` → no-op, return early
   - Emit state with `isDeterminingLocation: true`
   - Call `determineAppointmentLocationUseCase(DetermineAppointmentLocationParams(...))`
   - On success → emit `AppointmentCreatePopulateLocation(response.appointmentLocation)`, then emit state with `isDeterminingLocation: false`
   - On failure → emit state with `isDeterminingLocation: false` (location unchanged)
   - Guard against stale calls: check that the selected type has not changed after `await` before applying the result

4. **Modify `selectAppointmentType()`**:
   - Remove the existing `emit(AppointmentCreatePopulateLocation(type?.locationHardCode))` call
   - After updating state with the new type, call `_determineLocation()`
   - If `type == null` → skip `_determineLocation()`, ensure `isDeterminingLocation` is false

5. **Modify `customerChanged()`**:
   - After updating state with the new customer, call `_determineLocation()` if `selectedAppointmentType != null`

6. **Modify `ticketChanged()`**:
   - After updating state with the new ticket, call `_determineLocation()` if `selectedAppointmentType != null`

7. **Wire DI** in `appointment_create_page.dart`:
   - Resolve `DetermineAppointmentLocationUseCase` from the service locator (follow existing use case injection pattern in that file)

## File Ownership

- `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart`
- `syncro-flutter/lib/features/appointments/appointment_create/domain/appointment_create_state.dart`
- `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart`

## Notes

- The old `emit(AppointmentCreatePopulateLocation(type?.locationHardCode))` in `selectAppointmentType` must be REMOVED — the `determine_location` API call is the replacement. Do not keep both.
- Rapid changes: if the user changes customer and ticket in quick succession, stale API responses must be discarded. A simple approach is to snapshot `selectedAppointmentType` before the `await` and compare after; if it changed, discard the result.

## Success Criteria

- [ ] `AppointmentCreateCubit` constructor accepts `DetermineAppointmentLocationUseCase`
- [ ] Selecting an appointment type calls `determineLocation` and populates location on success
- [ ] Changing customer (when type is already selected) triggers `determineLocation`
- [ ] Changing ticket (when type is already selected) triggers `determineLocation`
- [ ] Setting type to null does NOT call `determineLocation`
- [ ] `isDeterminingLocation: true` is emitted while the API call is in progress
- [ ] On API failure, location value is preserved and `isDeterminingLocation` resets to `false`
- [ ] Old `locationHardCode`-based `PopulateLocation` call is removed from `selectAppointmentType`
- [ ] DI is wired in `appointment_create_page.dart`
- [ ] `flutter analyze` passes with no new warnings
