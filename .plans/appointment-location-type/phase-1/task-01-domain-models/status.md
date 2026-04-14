# Status: Domain Models & Params

**Task**: phase-1/task-01-domain-models
**Plan**: appointment-location-type
**Status**: complete
**Started**: 2026-04-14
**Completed**: 2026-04-14
**Agent**: claude-sonnet-4-6
**PR**: N/A

## Notes

### Adaptations
- **`officePresetLocation` kept (not removed)**: The static `officePresetLocation = 'US'` constant was kept in `AppointmentTypeOption` to avoid compile errors. Cubits reference it via `AppointmentTypeOption.officePresetLocation`. Removal deferred to task-04 where cubits are updated to use `type.locationHardCode ?? ''`.
- **`appointment_text_fields.dart` patched early**: This file (owned by task-04) has an exhaustive switch on `AppointmentLocationMode`. Adding `manual` to the enum required patching it in task-01 to restore compilation. The fix is minimal (just adds the `manual` case — same UI as `preset`).
- **Pre-existing test bugs fixed**: `update_appointment_params_test.dart` had wrong assertions (`expected 'id'` instead of `123`). `get_appointment_by_id_usecase_test.dart` props list was missing `appointmentLocationType` and `appointmentTypeId`. Both were pre-existing failures fixed here.

### Commit
`59290b8a` — feat(SE-11616): domain models — add manualEntry, locationType, appointmentLocationType params
