# Plan: Appointment Determine Location

**Status**: complete
**Created**: 2026-04-27
**Last Updated**: 2026-04-27
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: TBD (requires JIRA ticket)

## Objective

Auto-fill the `location` field in Create and Edit Appointment forms by calling `GET /appointment_types/:id/determine_location?customer_id=:customer_id&ticket_id=:ticket_id` whenever `appointment_type`, `customer`, or `ticket` changes. The BE response `{ "appointment_location": "US" }` provides the value to populate the field.

## Scope

### In Scope
- New endpoint: `GET /api/v1/appointment_types/:id/determine_location` with optional query params `customer_id` and `ticket_id`
- New `DetermineAppointmentLocationParams` and `DetermineAppointmentLocationResponse` domain models
- New `DetermineAppointmentLocationUseCase`
- New `determineLocation(params)` repository method (interface + impl)
- `AppointmentCreateCubit`: call `determineLocation` when:
  - `selectAppointmentType()` is called with a non-null type
  - `customerChanged()` is called and a type is already selected
  - `ticketChanged()` is called and a type is already selected
- `AppointmentEditCubit`: call `determineLocation` when:
  - `selectAppointmentType()` is called with a non-null type (customer and ticket are read-only in edit)
- `isDeterminingLocation: bool` loading flag in both loaded states
- UI: disable location field and show loading indicator while `isDeterminingLocation` is true
- On success: populate location field with `appointment_location` value from response
- On failure: keep existing location value, reset loading flag
- DI wiring: `DetermineAppointmentLocationUseCase` injected in create and edit pages

### Out of Scope
- Changing `AppointmentLocationMode` validation logic (address/phone/url/manual remain unchanged)
- Changing `appointment_location_type` in create/update payloads (covered by `appointment-location-type` plan)
- Changes to `AppointmentTypesCubit` itself
- Geocoding or address autocomplete for the `address` location mode
- Any changes to the appointment list/home feature

## Relationship with appointment-location-type plan

This plan is the natural extension of `appointment-location-type` (explicitly left out of scope there as "a separate feature"). The previous plan:
- Built `AppointmentTypesCubit` and the `/appointment_types` endpoint — reused here, not modified
- Added `appointment_location_type` (the TYPE field) to create/update payloads — not touched here
- Left `locationHardCode` as hardcoded `'US'` for `pre_defined` types — replaced here by `determine_location`

No file conflicts. The `selectAppointmentType()` methods in both cubits are extended, not replaced.

## Kill Criteria

- BE confirms `determine_location` endpoint does not exist or has a different URL/shape
- BE changes the response field name from `appointment_location`
- `ticketId` is unavailable in the Edit cubit state and cannot be obtained from the `Appointment` model (see Risks) — task-03 needs rework

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Foundation | task-01 | None | Domain models, networking, use case, repository |
| 2 | State | task-02, task-03 | Phase 1 | Cubit integration (tasks are parallel — different cubits) |
| 3 | Presentation | task-04 | Phase 2 | Loading state UI on location field |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| phase-1/task-01-domain-networking | Domain & Networking | 1 | not-started | — |
| phase-2/task-02-create-cubit | Create Cubit Integration | 2 | not-started | phase-1/task-01-domain-networking |
| phase-2/task-03-edit-cubit | Edit Cubit Integration | 2 | not-started | phase-1/task-01-domain-networking |
| phase-3/task-04-presentation | Presentation — Loading State | 3 | not-started | phase-2/task-02-create-cubit, phase-2/task-03-edit-cubit |

## Branch Convention

Work on the feature branch linked to the JIRA ticket (TBD). Single branch convention: one branch per JIRA ticket, no per-task branches.

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/core/networking/commons/app_requests_appointments.dart` | New `determineAppointmentLocation` endpoint |
| `syncro-flutter/lib/features/appointments/core/domain/appointments_repository.dart` | New `determineLocation()` method |
| `syncro-flutter/lib/features/appointments/core/infrastructure/appointments_repository_impl.dart` | Implement `determineLocation()` |
| `syncro-flutter/lib/features/appointments/core/domain/use_cases/determine_appointment_location_use_case.dart` | New use case (file to create) |
| `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart` | Call use case on type/customer/ticket change |
| `syncro-flutter/lib/features/appointments/appointment_create/domain/appointment_create_state.dart` | Add `isDeterminingLocation` flag |
| `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` | Call use case on type change |
| `syncro-flutter/lib/features/appointments/appointment_edit/domain/appointment_edit_state.dart` | Add `isDeterminingLocation` flag |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_view.dart` | Loading state on location field |
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_view.dart` | Loading state on location field |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart` | DI wiring |
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart` | DI wiring |

## Risks

1. **ticketId in Edit state** — `AppointmentEditLoaded` currently has `ticketNumber: String?` but NOT `ticketId: int?`. The endpoint needs a ticket ID. Verify if the `Appointment` model from BE includes a `ticketId` field to store in the edit state. This is a BLOCKER for task-03.
2. **locationHardCode replacement** — Current code in `selectAppointmentType` emits `PopulateLocation(type.locationHardCode)` for `pre_defined` types. After this plan, `determine_location` replaces that logic. Confirm with QA that the new API-driven behavior is tested end-to-end.
3. **Rapid field changes** — If the user changes customer and ticket in quick succession, multiple concurrent API calls fire. Consider a "cancel-on-new-call" pattern (discard stale responses). Evaluate during task-02 implementation.
4. **Optional params** — When neither `customer_id` nor `ticket_id` is available (standalone appointment creation), the call is made without those params. Verify with BE that this is a valid request.

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] Selecting an appointment type in Create form calls `determine_location` with type ID + available customer/ticket IDs
- [ ] Changing customer in Create form (when type is selected) triggers a new `determine_location` call
- [ ] Changing ticket in Create form (when type is selected) triggers a new `determine_location` call
- [ ] Selecting an appointment type in Edit form calls `determine_location` with type ID + available IDs
- [ ] The `appointment_location` value from the API response auto-fills the `location` field
- [ ] Location field shows a loading indicator while `determine_location` is in progress
- [ ] Location field is disabled during the API call
- [ ] On API failure, existing location value is preserved and loading is reset
- [ ] Setting appointment type to null does NOT call `determine_location`
- [ ] `customer_id` and `ticket_id` query params are omitted if they are null
- [ ] All existing appointment-related tests pass
- [ ] `flutter analyze` passes with no new warnings

## References

- **JIRA**: TBD
- **Related Plans**: `appointment-location-type` (completed — predecessor, adds `appointment_location_type` to create/update payloads)
