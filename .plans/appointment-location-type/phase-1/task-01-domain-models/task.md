# Task: Domain Models & Params

**Plan**: appointment-location-type
**Phase**: 1
**Task ID**: task-01
**Task Path**: phase-1/task-01-domain-models
**Depends On**: None
**JIRA**: SE-11616

## Objective

Fix the `AppointmentLocationType` enum (add `manualEntry`), update `AppointmentTypeOption` to carry a `locationType` field from the API and derive `locationMode` from it, add `AppointmentLocationMode.manual`, and add `appointmentLocationType` to both create and update params.

## Context

**Current state:**
- `AppointmentLocationType` (lib/features/appointments/core/domain/appointment_location_type.dart) has 3 values: `customer`, `shop`, `preDefined`. Missing `manualEntry`.
- `AppointmentTypeOption` (lib/features/ticket/ticket_home/domain/appointment_type_option.dart) has `id` and `name`. The `locationMode` getter is a heuristic based on name keywords (`phone`, `remote`, `our office`). This needs to become data-driven from a new `locationType` field.
- `AppointmentLocationMode` (same file) has: `address`, `phone`, `url`, `preset`. A new `manual` value is needed for `manual_entry` types that accept free-form text without geocoding validation.
- `CreateAppointmentParams` and `UpdateAppointmentParams` send `appointmentTypeId` but not `appointmentLocationType`.

**BE enum:** `location_type: {customer: 0, shop: 1, pre_defined: 2, manual_entry: 3}`

**API field name in JSON:** `"location_type"` per appointment type (verify with actual response).

**locationMode mapping:**
```
AppointmentLocationType.customer    → AppointmentLocationMode.address
AppointmentLocationType.shop        → AppointmentLocationMode.address
AppointmentLocationType.preDefined  → AppointmentLocationMode.preset
AppointmentLocationType.manualEntry → AppointmentLocationMode.manual
null                                → AppointmentLocationMode.address  (default, unchanged)
```

**AppointmentLocationMode.manual** behavior:
- `locationLabel`: `'Location'`
- `validationError`: `'Please enter a location'`
- `isValid(value)`: `value.trim().isNotEmpty` (no geocoding API call — cubit already skips API for any mode != address)

**Params change:** Both `CreateAppointmentParams` and `UpdateAppointmentParams` get an optional `String? appointmentLocationType` field. In `toMap()`:
```dart
if (appointmentLocationType != null) 'appointment_location_type': appointmentLocationType,
```

**`location_hard_code`:** Each appointment type has a `location_hard_code: String?` field. When the type is `pre_defined`, this value is used to auto-fill the location field (e.g., `"Phone Call"`, `"Remote Support Link: (edit me)"`). For `customer` and `shop` types it is `null`. The static `officePresetLocation = 'US'` constant must be **removed** and replaced by this instance field.

**Wire value in cubits (done in task-04):** The raw string value to send is `selectedAppointmentType?.locationType?.toApiString`. Add a `toApiString` getter to `AppointmentLocationType` that returns the snake_case BE string (`'customer'`, `'shop'`, `'pre_defined'`, `'manual_entry'`).

**`selectAppointmentType()` preset logic (done in task-04):** Replace `AppointmentTypeOption.officePresetLocation` with `type.locationHardCode ?? ''`.

## Before You Start

- [ ] Read `lib/features/appointments/core/domain/appointment_location_type.dart`
- [ ] Read `lib/features/ticket/ticket_home/domain/appointment_type_option.dart`
- [ ] Read `lib/features/appointments/appointment_create/domain/create_appointment_params.dart`
- [ ] Read `lib/features/appointments/appointment_edit/domain/update_appointment_params.dart`
- [ ] Check existing tests for these files: `test/features/appointments/`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/core/domain/appointment_location_type.dart` | modify | Add `manualEntry`, add `toApiString()` getter |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/appointment_type_option.dart` | modify | Add `AppointmentLocationMode.manual`; add `locationType` field to `AppointmentTypeOption`; update `fromJson`; replace name-based `locationMode` with `locationType`-based switch |
| `syncro-flutter/lib/features/appointments/appointment_create/domain/create_appointment_params.dart` | modify | Add `String? appointmentLocationType`; include in `toMap()` |
| `syncro-flutter/lib/features/appointments/appointment_edit/domain/update_appointment_params.dart` | modify | Add `String? appointmentLocationType`; include in `toMap()` |

### Do NOT Modify

- `syncro-flutter/lib/features/appointments/core/domain/appointments_repository.dart` — owned by phase-1/task-02-networking-repository
- `syncro-flutter/lib/features/appointments/core/infrastructure/appointments_repository_impl.dart` — owned by phase-1/task-02-networking-repository
- Any cubit files — owned by phase-2/task-03 and phase-3/task-04
- Any view/page files — owned by phase-3/task-04

## Implementation Steps

### Step 1: Fix AppointmentLocationType enum

In `appointment_location_type.dart`:

1. Add `manualEntry` to the enum values.
2. Update `fromString()`:
   ```dart
   case 'manual_entry':
     return AppointmentLocationType.manualEntry;
   ```
3. Update `displayName` getter to include `manualEntry → 'Manual Entry'`.
4. Add `toApiString()` getter:
   ```dart
   String get toApiString => switch (this) {
     AppointmentLocationType.customer    => 'customer',
     AppointmentLocationType.shop        => 'shop',
     AppointmentLocationType.preDefined  => 'pre_defined',
     AppointmentLocationType.manualEntry => 'manual_entry',
   };
   ```

### Step 2: Add AppointmentLocationMode.manual

In `appointment_type_option.dart`, add `manual` to the `AppointmentLocationMode` enum:
```dart
/// Free-form text input — no geocoding validation, no autocomplete.
manual;
```

Update all switch expressions that are exhaustive on `AppointmentLocationMode` (check for compile errors — likely `locationLabel`, `validationError`, `isValid`):
- `locationLabel`: `manual → 'Location'`
- `validationError`: `manual → 'Please enter a location'`
- `isValid`: `manual → value.trim().isNotEmpty`

### Step 3: Update AppointmentTypeOption

Each appointment type returned by `/appointment_types` includes a `location_hard_code` field — a string used as the preset location value when `pre_defined` is selected. Replace the static `officePresetLocation = 'US'` constant with an instance field.

1. Remove `static const String officePresetLocation = 'US';`
2. Add instance fields:
   ```dart
   final AppointmentLocationType? locationType;
   final String? locationHardCode;  // preset value for pre_defined types
   ```
3. Update constructor to include both fields.
4. Update `fromJson`:
   ```dart
   locationType: AppointmentLocationType.fromString(
     map['location_type'] as String?,
   ),
   locationHardCode: map['location_hard_code'] as String?,
   ```
5. Update `toMap()`:
   ```dart
   if (locationType != null) 'location_type': locationType!.toApiString,
   if (locationHardCode != null) 'location_hard_code': locationHardCode,
   ```
6. Update `props` to include both `locationType` and `locationHardCode`.
7. Replace the name-based `locationMode` getter with:
   ```dart
   AppointmentLocationMode get locationMode {
     if (locationType == null) return AppointmentLocationMode.address;
     return switch (locationType!) {
       AppointmentLocationType.customer    => AppointmentLocationMode.address,
       AppointmentLocationType.shop        => AppointmentLocationMode.address,
       AppointmentLocationType.preDefined  => AppointmentLocationMode.preset,
       AppointmentLocationType.manualEntry => AppointmentLocationMode.manual,
     };
   }
   ```

### Step 4: Update CreateAppointmentParams

1. Add field: `final String? appointmentLocationType;`
2. Add to constructor as optional named param.
3. Add to `toMap()`:
   ```dart
   if (appointmentLocationType != null)
     'appointment_location_type': appointmentLocationType,
   ```
4. Update `toString()` to include the field.

### Step 5: Update UpdateAppointmentParams

Same changes as Step 4, applied to `UpdateAppointmentParams`.

## Testing

- [ ] `AppointmentLocationType.fromString('manual_entry')` returns `AppointmentLocationType.manualEntry`
- [ ] `AppointmentLocationType.manualEntry.toApiString` returns `'manual_entry'`
- [ ] `AppointmentLocationType.preDefined.toApiString` returns `'pre_defined'`
- [ ] `AppointmentTypeOption` with `location_type: 'pre_defined'` has `locationMode == AppointmentLocationMode.preset`
- [ ] `AppointmentTypeOption` with `location_type: 'manual_entry'` has `locationMode == AppointmentLocationMode.manual`
- [ ] `AppointmentTypeOption` with `location_type: null` has `locationMode == AppointmentLocationMode.address`
- [ ] `CreateAppointmentParams.toJson()` includes `appointment_location_type` when set, omits when null
- [ ] `UpdateAppointmentParams.toJson()` includes `appointment_location_type` when set, omits when null
- [ ] Existing tests in `test/features/appointments/` still pass
- [ ] `flutter analyze` passes with no new warnings

## Documentation / KB Updates

- [ ] No KB doc updates required for this task — changes are additive domain model fixes

## Completion Criteria

- [ ] `AppointmentLocationType` has 4 values including `manualEntry`
- [ ] `AppointmentTypeOption.locationType` field present, parsed from `"location_type"` JSON key
- [ ] `locationMode` derives from `locationType`, not name keywords
- [ ] `AppointmentLocationMode.manual` is exhaustively handled in all switch expressions
- [ ] Both params classes include optional `appointmentLocationType` field that serializes to `"appointment_location_type"` in JSON
- [ ] All tests pass
- [ ] `flutter analyze` clean
