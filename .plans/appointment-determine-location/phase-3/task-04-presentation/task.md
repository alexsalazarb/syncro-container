# Task 04: Presentation — Loading State

**Plan**: appointment-determine-location
**Phase**: 3 — Presentation
**Status**: not-started
**Depends on**: phase-2/task-02-create-cubit, phase-2/task-03-edit-cubit

## Objective

Update the location field UI in both Create and Edit forms to reflect the `isDeterminingLocation` loading state: show a loading indicator and disable the field while the API call is in flight.

## Steps

1. **Locate location field** — Find where the location `CustomTextField` / `CustomTextFieldWithHint` is built inside `AppointmentTextFields` (or inline in the views). Confirm whether it's a shared widget or duplicated per view.

2. **Pass `isDeterminingLocation`** from the BlocBuilder state down to the location field widget:
   ```dart
   // In BlocBuilder where AppointmentCreateLoaded is handled:
   AppointmentTextFields(
     ...
     isDeterminingLocation: stateLoaded.isDeterminingLocation,
   )
   ```

3. **While `isDeterminingLocation == true`**:
   - Add a `CircularProgressIndicator` as `suffixIcon` on the location text field
   - Set `enabled: false` on the field to prevent user edits during the call

4. **While `isDeterminingLocation == false`**:
   - Remove the indicator (restore normal suffix icon if the field has one)
   - Re-enable the text field

5. **Apply to both forms**:
   - `appointment_create_view.dart`
   - `appointment_edit_view.dart`

6. **No layout shift**: ensure the loading indicator does not change the field height or cause surrounding fields to move.

## File Ownership

- `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_view.dart`
- `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_view.dart`
- (possibly) shared widget used by both views for the location text field

## Success Criteria

- [ ] Location field shows a `CircularProgressIndicator` as suffix while `isDeterminingLocation` is true
- [ ] Location field is disabled (not editable) while loading
- [ ] Loading indicator disappears and field re-enables after the API call completes (success or failure)
- [ ] No layout shift when the loading indicator appears or disappears
- [ ] All other form fields are unaffected visually
- [ ] Change applies to both Create and Edit forms
- [ ] `flutter analyze` passes with no new warnings
