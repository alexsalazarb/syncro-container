# Plan: Appointment Location Type — Create & Update Support

**Status**: complete
**Created**: 2026-04-14
**Last Updated**: 2026-04-14
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: feature/SE-11616

## Objective

Add `appointment_location_type` to create and update appointment API calls, driven by a new dedicated `/appointment_types` endpoint that replaces the existing `TicketsSettings` usage for appointment type selection.

## Scope

### In Scope
- Fix `AppointmentLocationType` enum: add `manualEntry` value
- Update `AppointmentTypeOption`: add `locationType` field from API, derive `locationMode` from `locationType` instead of name keywords
- Add `AppointmentLocationMode.manual` for free-form location entry without geocoding validation
- New endpoint: `GET /api/v1/appointment_types` — endpoint, deserializer, use case, repository method
- New `AppointmentTypesCubit` — fetches/caches types from dedicated endpoint, page-level scoped
- Add `appointmentLocationType` to `CreateAppointmentParams` and `UpdateAppointmentParams`
- Cubits pass `appointmentLocationType` from selected type's `locationType` to params
- Replace `TicketsSettingsCubit` usage in `_showAppointmentTypeBottomSheet` and type selectors (both create and edit views) with `AppointmentTypesCubit`
- DI wiring: page-level `AppointmentTypesCubit` in create and edit pages

### Out of Scope
- Changing the `TicketsSettingsCubit` itself or removing `appointmentTypes` from `TicketsSettings` — other features still depend on it
- Handling pre_defined location preset value from the API — current 'US' hardcode is kept; verify with BE
- Address autocomplete behavior changes for `customer` / `shop` location types (auto-fill from ticket/customer data is a separate feature)
- Any changes to the appointment list/home feature

## Kill Criteria

- BE confirms that `/appointment_types` endpoint does not include a `location_type` field per item — task-02 and task-04 need redesign
- BE changes the `appointment_location_type` field name in create/update payloads
- The `AppointmentTypeOption` model needs to be relocated to the appointments feature — coordinate with any concurrent work touching that file

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Foundation | task-01, task-02 | None (task-02 after task-01) | Domain models, params, networking, repository |
| 2 | State | task-03 | Phase 1 | AppointmentTypesCubit shared state layer |
| 3 | Presentation | task-04 | Phase 2 | Cubit updates, view wiring, DI |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| phase-1/task-01-domain-models | Domain Models & Params | 1 | complete | — |
| phase-1/task-02-networking-repository | Networking & Repository | 1 | complete | phase-1/task-01-domain-models |
| phase-2/task-03-appointment-types-cubit | AppointmentTypesCubit | 2 | complete | phase-1/task-02-networking-repository |
| phase-3/task-04-presentation | Presentation & DI | 3 | complete | phase-2/task-03-appointment-types-cubit |

## Branch Convention

Work on the existing feature branch: `feature/SE-11616` (no per-task branches — single JIRA ticket convention for this project).

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/features/appointments/core/domain/appointment_location_type.dart` | Enum fix (add manualEntry) |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/appointment_type_option.dart` | Add locationType, update locationMode |
| `syncro-flutter/lib/core/networking/commons/app_requests_appointments.dart` | New getAppointmentTypes endpoint |
| `syncro-flutter/lib/features/appointments/core/domain/appointments_repository.dart` | Add getAppointmentTypes() |
| `syncro-flutter/lib/features/appointments/core/infrastructure/appointments_repository_impl.dart` | Implement getAppointmentTypes() |
| `syncro-flutter/lib/features/appointments/appointment_create/domain/create_appointment_params.dart` | Add appointmentLocationType |
| `syncro-flutter/lib/features/appointments/appointment_edit/domain/update_appointment_params.dart` | Add appointmentLocationType |
| `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart` | Pass appointmentLocationType |
| `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` | Pass appointmentLocationType |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_view.dart` | Replace TicketsSettingsCubit usage |
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_view.dart` | Replace TicketsSettingsCubit usage |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart` | DI wiring |
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart` | DI wiring |

## Risks

- **`location_type` field name** — assumed to be `"location_type"` in the `/appointment_types` JSON; verify against actual BE response before task-02
- **pre_defined preset value** — currently hardcoded as `'US'` in `AppointmentTypeOption.officePresetLocation`; the BE may return a different value or include it in the response. Flag for QA to verify.
- **TicketsSettings still returns appointmentTypes** — the old source remains live. After this plan, two sources of truth exist. This is intentional short-term; clean-up is out of scope.
- **`AppointmentTypeOption` file location** — currently in `ticket_home/domain/`, which is odd for an appointments feature. Moving it is out of scope but worth noting for future housekeeping.

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] `appointment_location_type` is present in create appointment request payload when a type with a `locationType` is selected
- [ ] `appointment_location_type` is present in update appointment request payload when a type with a `locationType` is selected
- [ ] Appointment type bottom sheet in create and edit flows loads from `/appointment_types`, not from `tickets/settings`
- [ ] `AppointmentLocationType.manualEntry` exists and `Appointment.fromJson` correctly parses `"manual_entry"` strings
- [ ] `manual_entry` location type skips geocoding validation (accepts any non-empty text)
- [ ] `pre_defined` location type auto-fills location field (existing behavior preserved)
- [ ] All existing appointment-related tests pass
- [ ] `flutter analyze` passes with no new warnings

## References

- **JIRA**: SE-11616
- **Related Plans**: None
