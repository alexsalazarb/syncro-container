# Status: Presentation & DI

**Task**: phase-3/task-04-presentation
**Plan**: appointment-location-type
**Status**: complete
**Started**: 2026-04-14
**Completed**: 2026-04-14
**Agent**: claude-sonnet-4-6
**PR**: N/A
**Commit**: 537ff414

## Notes

- Replaced `TicketsSettingsCubit` with `AppointmentTypesCubit` in both create and edit views (bottom sheet + BlocBuilder type selectors)
- Added `appointmentLocationType` to `CreateAppointmentParams` and `UpdateAppointmentParams` in both cubits
- Updated `selectAppointmentType()` in both cubits: uses `type?.locationHardCode ?? ''` instead of removed static `officePresetLocation`
- Page files updated to `MultiBlocProvider` providing `AppointmentTypesCubit` alongside the existing cubit; removed `TicketsSettingsCubit.load()` prefetch calls
- `AppointmentTypeOption` needed explicit import in both views (not re-exported from `appointments.dart` barrel)
