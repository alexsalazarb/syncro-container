# task-01: Extend CrudAppointmentBaseCubit with Abstract Interface + Shared Methods

**Task ID**: task-01  
**Plan**: unify-appointment-crud-cubit  
**Depends On**: —  
**Branch**: `plan/unify-appointment-crud-cubit/task-01-extend-base-cubit`

---

## Objective

Extend `CrudAppointmentBaseCubit<T>` with:
1. `DetermineAppointmentLocationUseCase` constructor parameter
2. Ten abstract methods for state manipulation (delegated to child cubits)
3. The three shared concrete methods: `selectAppointmentType()`, `_determineLocation()`, `sendCustomerEmailChanged()`

After this task the class compiles but is abstract — child cubits still have their own duplicate implementations (those are removed in task-02 and task-03).

---

## File Ownership

**Modifies**:
- `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_crud_base_cubit.dart`

**Do NOT Modify** (owned by task-02/task-03):
- `appointment_create_cubit.dart`
- `appointment_edit_cubit.dart`

---

## Steps

### 1. Add required imports to `appointment_crud_base_cubit.dart`

```dart
import 'package:syncro/core/services/analytics/analytics_manager.dart';
import 'package:syncro/core/utils/logger.dart';
import 'package:syncro/features/appointments/appointments.dart';
import 'package:syncro/features/customers/domain/customer.dart';
import 'package:syncro/features/ticket/ticket_home/domain/appointment_type_option.dart';
import 'package:syncro/features/appointments/appointment_create/domain/determine_appointment_location_usecase.dart';
import 'package:syncro/features/appointments/appointment_create/domain/determine_appointment_location_params.dart';
```

### 2. Add `determineAppointmentLocationUseCase` to the base constructor

```dart
class CrudAppointmentBaseCubit<T> extends Cubit<T> {
  CrudAppointmentBaseCubit(
    super.initialState, {
    required this.determineAppointmentLocationUseCase,
  });

  final DetermineAppointmentLocationUseCase determineAppointmentLocationUseCase;
  // ... existing fields
```

### 3. Add abstract methods for state delegation

```dart
  // ── Abstract interface ──────────────────────────────────────────────────────

  /// Returns the current loaded state, or null if the cubit is not in a loaded state.
  T? get loadedStateOrNull;

  /// Builds the PopulateLocation transient state for this cubit's state hierarchy.
  T buildPopulateLocationState(String location);

  /// Returns [loaded] with [isDeterminingLocation] set to [value].
  T withDeterminingLocation(T loaded, {required bool value});

  /// Returns [loaded] with [locationFromApi] set to [location].
  T withLocationFromApi(T loaded, String location);

  /// Returns [loaded] with [selectedAppointmentType] set to [type].
  T withSelectedType(T loaded, AppointmentTypeOption type);

  /// Returns [loaded] with [selectedAppointmentType] and [locationFromApi] cleared.
  T withClearedTypeAndLocation(T loaded);

  /// Returns [loaded] with [sendCustomerEmail] set to [value].
  T withSendCustomerEmail(T loaded, bool value);

  /// Returns the selected appointment type from [loaded], or null.
  AppointmentTypeOption? selectedTypeOf(T loaded);

  /// Returns the customer from [loaded], or null.
  Customer? customerOf(T loaded);

  /// Returns the ticket ID from [loaded], or null.
  /// Create: `ticketData?.id`. Edit: `ticketId`.
  int? ticketIdOf(T loaded);
```

### 4. Add the three shared concrete methods

```dart
  // ── Shared concrete methods ─────────────────────────────────────────────────

  void sendCustomerEmailChanged(bool? value) {
    final loaded = loadedStateOrNull;
    if (loaded == null) return;
    analyticsManager.logEvent(
      value == true
          ? AnalyticsEventEnum.apptSendEmail
          : AnalyticsEventEnum.apptNoSendEmail,
    );
    safeEmit(withSendCustomerEmail(loaded, value ?? false));
  }

  Future<void> selectAppointmentType(AppointmentTypeOption? type) async {
    final loaded = loadedStateOrNull;
    if (loaded == null) return;

    final newLoaded = type == null
        ? withClearedTypeAndLocation(loaded)
        : withSelectedType(loaded, type);
    safeEmit(newLoaded);

    if (type != null) {
      await _determineLocation();
    }
  }

  Future<void> _determineLocation() async {
    final currentLoaded = loadedStateOrNull;
    if (currentLoaded == null) return;

    final selectedType = selectedTypeOf(currentLoaded);
    if (selectedType == null) return;

    safeEmit(withDeterminingLocation(currentLoaded, value: true));

    final result = await determineAppointmentLocationUseCase(
      DetermineAppointmentLocationParams(
        appointmentTypeId: selectedType.id,
        customerId: customerOf(currentLoaded)?.id,
        ticketId: ticketIdOf(currentLoaded),
      ),
    );

    // Stale-check: abort if the selected type changed while awaiting.
    final latestLoaded = loadedStateOrNull;
    if (latestLoaded == null) return;
    if (selectedTypeOf(latestLoaded)?.id != selectedType.id) return;

    result.fold(
      (failure) {
        safeEmit(withDeterminingLocation(latestLoaded, value: false));
      },
      (response) {
        safeEmit(buildPopulateLocationState(response.appointmentLocation));
        safeEmit(
          withLocationFromApi(
            withDeterminingLocation(latestLoaded, value: false),
            response.appointmentLocation,
          ),
        );
      },
    );
  }
```

### 5. Run format and analyze

```bash
fvm dart format syncro-flutter/lib/features/appointments/appointment_create/application/appointment_crud_base_cubit.dart
fvm flutter analyze syncro-flutter/lib/features/appointments/
```

Expect analyzer errors for child cubits not yet implementing the abstract methods — those are fixed in tasks 02 and 03.

---

## Testing

No new tests for task-01 alone — the base class is abstract. Tests come in task-03.

Verify the file compiles by checking analyzer output for `appointment_crud_base_cubit.dart` specifically.

---

## Acceptance Criteria

- [ ] `appointment_crud_base_cubit.dart` compiles with the new abstract methods and concrete implementations
- [ ] `_determineLocation()`, `selectAppointmentType()`, `sendCustomerEmailChanged()` exist in the base class
- [ ] `determineAppointmentLocationUseCase` is a field on the base class
- [ ] `fvm dart format` produces no changes on the file
