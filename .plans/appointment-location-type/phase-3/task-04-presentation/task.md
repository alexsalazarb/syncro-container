# Task: Presentation & DI

**Plan**: appointment-location-type
**Phase**: 3
**Task ID**: task-04
**Task Path**: phase-3/task-04-presentation
**Depends On**: phase-2/task-03-appointment-types-cubit
**JIRA**: SE-11616

## Objective

Wire `appointmentLocationType` into `CreateAppointmentParams` / `UpdateAppointmentParams` from the cubits, replace `TicketsSettingsCubit` with `AppointmentTypesCubit` in both appointment views (bottom sheet + type selector widgets), and update both pages to provide `AppointmentTypesCubit` instead of pre-fetching via `TicketsSettingsCubit`.

## Context

**Three change areas:**

**1. Cubit params wiring (appointment_create_cubit + appointment_edit_cubit)**

In `validateAndSaveAppointment()` (create) and `validateAndUpdateAppointment()` (edit), the params are built. Add:
```dart
appointmentLocationType: stateLoaded.selectedAppointmentType?.locationType?.toApiString,
```
Both methods already access `stateLoaded.selectedAppointmentType`. The `locationType` field and `toApiString` getter are added in task-01.

**2. View changes (appointment_create_view + appointment_edit_view)**

`_showAppointmentTypeBottomSheet` currently does:
```dart
final ticketsSettings = await context.read<TicketsSettingsCubit>().load();
final types = ticketsSettings?.appointmentTypes ?? [];
```
Replace with:
```dart
final types = await context.read<AppointmentTypesCubit>().load();
```

`_AppointmentTypeSelector` and `_EditAppointmentTypeSelector` currently use `BlocBuilder<TicketsSettingsCubit>` to get the types list for displaying the selected name. Replace with `BlocBuilder<AppointmentTypesCubit>`:
```dart
BlocBuilder<AppointmentTypesCubit, AppointmentTypesState>(
  builder: (context, typesState) {
    final types = typesState is AppointmentTypesLoaded ? typesState.types : <AppointmentTypeOption>[];
    final selectedName = types
        .where((t) => t.id == selectedTypeId)
        .map((t) => t.name)
        .firstOrNull;
    return CustomItemSelector(...);
  },
)
```

Also remove `import 'package:syncro/features/ticket/shared/application/tickets_settings_cubit/tickets_settings_cubit.dart';` from both views (if no longer used).

**3. Page-level DI (appointment_create_page + appointment_edit_page)**

Both pages currently do:
```dart
context.read<TicketsSettingsCubit>().load();  // prefetch
return AppointmentCreateCubit(...);           // in BlocProvider.create
```

Replace the prefetch with `AppointmentTypesCubit` creation. Use `MultiBlocProvider` so both `AppointmentCreateCubit` (or `AppointmentEditCubit`) and `AppointmentTypesCubit` are in the widget tree:

```dart
return MultiBlocProvider(
  providers: [
    BlocProvider<AppointmentTypesCubit>(
      create: (context) => AppointmentTypesCubit(
        GetAppointmentTypesUseCase(context.read<AppointmentsRepository>()),
      ),
    ),
    BlocProvider<AppointmentCreateCubit>(
      create: (context) => AppointmentCreateCubit(
        // ... existing args unchanged
      ),
    ),
  ],
  child: AppScaffold(...),
);
```

**For `AppointmentEditPage`:** The `init()` method in `AppointmentEditView` currently reads:
```dart
final settings = await context.read<TicketsSettingsCubit>().load();
final types = settings?.appointmentTypes ?? [];
final selectedType = appointment.appointmentTypeId != null
    ? types.where((t) => t.id == appointment.appointmentTypeId).firstOrNull
    : null;
```
Replace with:
```dart
final types = await context.read<AppointmentTypesCubit>().load();
final selectedType = appointment.appointmentTypeId != null
    ? types.where((t) => t.id == appointment.appointmentTypeId).firstOrNull
    : null;
```

**Note on `TicketsSettingsCubit` removal from imports:** Do NOT remove `TicketsSettingsCubit` from the pages' imports if it is used elsewhere in the page (e.g., other widget reads). Check carefully before removing any import.

## Before You Start

- [ ] Confirm task-03 is complete (`AppointmentTypesCubit` exists)
- [ ] Confirm task-01 is complete (`AppointmentLocationType.toApiString` exists)
- [ ] Read `lib/features/appointments/appointment_create/presentation/appointment_create_page.dart`
- [ ] Read `lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart`
- [ ] Read `lib/features/appointments/appointment_create/presentation/appointment_create_view.dart`
- [ ] Read `lib/features/appointments/appointment_edit/presentation/appointment_edit_view.dart`
- [ ] Read `lib/features/appointments/appointment_create/application/appointment_create_cubit.dart`
- [ ] Read `lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart` | modify | Add `appointmentLocationType` to `CreateAppointmentParams` call |
| `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` | modify | Add `appointmentLocationType` to both `UpdateAppointmentParams` calls (there are two — preview and final) |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_view.dart` | modify | Replace `TicketsSettingsCubit` in `_showAppointmentTypeBottomSheet` and `_AppointmentTypeSelector` |
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_view.dart` | modify | Replace `TicketsSettingsCubit` in `_showAppointmentTypeBottomSheet`, `_EditAppointmentTypeSelector`, and `init()` method |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart` | modify | MultiBlocProvider wrapping, provide `AppointmentTypesCubit` |
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart` | modify | MultiBlocProvider wrapping, provide `AppointmentTypesCubit` |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/shared/application/tickets_settings_cubit/tickets_settings_cubit.dart` — not in scope; other features depend on it
- Any repository, use case, or domain model files — owned by earlier tasks

## Implementation Steps

### Step 1: Cubit — wire appointmentLocationType to params

In `appointment_create_cubit.dart`, find `validateAndSaveAppointment()`. Locate where `CreateAppointmentParams` is built and add:
```dart
appointmentLocationType: stateLoaded.selectedAppointmentType?.locationType?.toApiString,
```

In `appointment_edit_cubit.dart`, find `validateAndUpdateAppointment()`. There are **two** places where `UpdateAppointmentParams` is constructed (one for preview/early return and one for the actual update). Add `appointmentLocationType` to **both**.

### Step 2: Create view — replace TicketsSettingsCubit in bottom sheet

In `appointment_create_view.dart`:

1. Replace import:
   - Remove: `import 'package:syncro/features/ticket/shared/application/tickets_settings_cubit/tickets_settings_cubit.dart';`
   - Add: `import 'package:syncro/features/appointments/core/application/appointment_types_cubit.dart';`

2. In `_showAppointmentTypeBottomSheet()`:
   - Replace `context.read<TicketsSettingsCubit>().load()` with `context.read<AppointmentTypesCubit>().load()`
   - Remove `ticketsSettings?.appointmentTypes ?? []` — use the returned list directly

3. In `_AppointmentTypeSelector.build()`:
   - Replace `BlocBuilder<TicketsSettingsCubit, TicketsSettingsState>` with `BlocBuilder<AppointmentTypesCubit, AppointmentTypesState>`
   - Update the types extraction:
     ```dart
     final types = state is AppointmentTypesLoaded ? state.types : <AppointmentTypeOption>[];
     ```

### Step 3: Edit view — replace TicketsSettingsCubit in bottom sheet, selector, and init

In `appointment_edit_view.dart`:

1. Replace import (same as Step 2).

2. In `_showAppointmentTypeBottomSheet()`: same replacement as Step 2.

3. In `_EditAppointmentTypeSelector.build()`: same BlocBuilder replacement as Step 2.

4. In `init()` method: replace settings load with cubit load (see Context section above).

### Step 4: Create page — MultiBlocProvider

In `appointment_create_page.dart`:

1. Add import for `AppointmentTypesCubit` and `GetAppointmentTypesUseCase`.
2. Remove `context.read<TicketsSettingsCubit>().load()` call.
3. Wrap the existing `BlocProvider<AppointmentCreateCubit>` with `MultiBlocProvider` that also provides `AppointmentTypesCubit`.

### Step 5: Edit page — MultiBlocProvider

Same pattern as Step 4, applied to `appointment_edit_page.dart` with `AppointmentEditCubit`.

## Testing

- [ ] Manually test (or integration test): selecting an appointment type in create flow calls the `/appointment_types` endpoint (not `tickets/settings`)
- [ ] Selecting a `pre_defined` type auto-fills the location field
- [ ] Selecting a `manual_entry` type clears the location field; any text passes validation (no geocoding call)
- [ ] `appointment_location_type` appears in the create payload (verify via network logs or unit test on cubit)
- [ ] `appointment_location_type` appears in the update payload
- [ ] Selecting no appointment type: params do NOT include `appointment_location_type`
- [ ] Edit flow: existing appointment with a type pre-selects the correct type name from the `AppointmentTypesCubit` list
- [ ] Existing cubit tests for `appointment_create_cubit` and `appointment_edit_cubit` still pass
- [ ] `flutter analyze` clean with no unused import warnings

## Documentation / KB Updates

- [ ] No new KB doc required — change is a straightforward DI replacement following established patterns

## Completion Criteria

- [ ] `appointmentLocationType` passed to both `CreateAppointmentParams` and `UpdateAppointmentParams` in the respective cubits
- [ ] Both views use `AppointmentTypesCubit` exclusively for appointment type data (no `TicketsSettingsCubit` references for this purpose)
- [ ] Both pages provide `AppointmentTypesCubit` via `MultiBlocProvider`
- [ ] No `TicketsSettingsCubit.load()` calls in create/edit pages (removed)
- [ ] All acceptance criteria in `overview.md` verified
- [ ] All tests pass; `flutter analyze` clean
