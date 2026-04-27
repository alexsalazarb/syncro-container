# Task 03: Edit Cubit Integration

**Plan**: appointment-determine-location
**Phase**: 2 — State
**Status**: not-started
**Depends on**: phase-1/task-01-domain-networking

## Objective

Integrate `DetermineAppointmentLocationUseCase` into `AppointmentEditCubit` so that the `location` field is auto-filled via API when the user changes the appointment type.

## Steps

1. **BLOCKER — Verify ticket ID availability**: Check the `Appointment` model loaded in `getAppointment()`. If it includes a `ticketId: int?` field, store it in `AppointmentEditLoaded` (currently only `ticketNumber: String?` is stored). If the `Appointment` model does NOT include a ticket ID, document this gap and send `ticketId: null` in the request (the `customer_id` will still be sent).

2. **Inject use case** via constructor:
   ```dart
   AppointmentEditCubit({
     ...existing params...,
     required this.determineAppointmentLocationUseCase,
   })
   ```

3. **Add `isDeterminingLocation: bool` to `AppointmentEditLoaded`** (defaults to `false`).

4. **Create private helper `_determineLocation()`** (same pattern as Create):
   - Read `selectedAppointmentType`, `customer?.id`, `ticketId` from current state
   - If `selectedAppointmentType == null` → no-op
   - Emit state with `isDeterminingLocation: true`
   - Call use case with optional `customerId`/`ticketId`
   - On success → emit `AppointmentEditPopulateLocation(response.appointmentLocation)`, then reset `isDeterminingLocation: false`
   - On failure → reset `isDeterminingLocation: false` (location unchanged)

5. **Modify `selectAppointmentType()`**:
   - Remove the existing `emit(AppointmentEditPopulateLocation(type?.locationHardCode))` call
   - After updating state with the new type, call `_determineLocation()`
   - If `type == null` → skip `_determineLocation()`

6. **Wire DI** in `appointment_edit_page.dart`:
   - Resolve `DetermineAppointmentLocationUseCase` from the service locator

## File Ownership

- `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart`
- `syncro-flutter/lib/features/appointments/appointment_edit/domain/appointment_edit_state.dart`
- `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart`
- (possibly) `Appointment` domain model if `ticketId` needs to be added

## Notes

- In Edit mode, customer and ticket are always read-only (loaded from the saved appointment). Only `selectAppointmentType()` is a trigger — no `customerChanged` or `ticketChanged` equivalents needed.
- Step 1 is a BLOCKER: resolve ticket ID availability before implementing `_determineLocation()`.
- The old `emit(AppointmentEditPopulateLocation(type?.locationHardCode))` must be REMOVED, same as in task-02.

## Success Criteria

- [ ] `AppointmentEditCubit` constructor accepts `DetermineAppointmentLocationUseCase`
- [ ] Changing appointment type in the edit form calls `determineLocation` and populates location on success
- [ ] `customer_id` from the loaded appointment is sent in the request
- [ ] `ticket_id` is sent if available from the `Appointment` model (or omitted if not)
- [ ] `isDeterminingLocation: true` is emitted while the API call is in progress
- [ ] On API failure, existing location value is preserved
- [ ] Old `locationHardCode`-based `PopulateLocation` call is removed from `selectAppointmentType`
- [ ] DI is wired in `appointment_edit_page.dart`
- [ ] `flutter analyze` passes with no new warnings
