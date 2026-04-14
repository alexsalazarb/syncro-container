# Task: AppointmentTypesCubit

**Plan**: appointment-location-type
**Phase**: 2
**Task ID**: task-03
**Task Path**: phase-2/task-03-appointment-types-cubit
**Depends On**: phase-1/task-02-networking-repository
**JIRA**: SE-11616

## Objective

Create `AppointmentTypesCubit` — a page-level cubit that fetches appointment types from the new `/appointment_types` endpoint, caches the result, and exposes it to the create/edit appointment views.

## Context

**Why a new cubit instead of reusing `TicketsSettingsCubit`:**
- `TicketsSettingsCubit` fetches the heavyweight `tickets/settings` endpoint; coupling appointment type selection to it creates unnecessary overhead and a cross-feature dependency.
- `AppointmentTypesCubit` has a single responsibility: load and cache `List<AppointmentTypeOption>` from the dedicated endpoint.

**Scope:** Page-level (not global). Both `AppointmentCreatePage` and `AppointmentEditPage` will each create their own instance. The cubit is short-lived and tied to the page's lifecycle.

**State design — keep it minimal:**
```dart
sealed class AppointmentTypesState extends Equatable {
  const AppointmentTypesState();
}

final class AppointmentTypesLoading extends AppointmentTypesState { ... }

final class AppointmentTypesLoaded extends AppointmentTypesState {
  final List<AppointmentTypeOption> types;
  ...
}

final class AppointmentTypesError extends AppointmentTypesState {
  final String message;
  ...
}
```

**Cubit behavior:**
- Constructor: immediately calls `_load()` internally (auto-fetch on creation).
- `_load()`: calls `GetAppointmentTypesUseCase`, emits `AppointmentTypesLoaded` on success, `AppointmentTypesError` on failure.
- `load()` public method: returns cached `List<AppointmentTypeOption>` if already loaded, awaits otherwise (mirrors `TicketsSettingsCubit.load()` for easy substitution).
- Use `safeEmit()` from `lib/core/utils/cubit_extension.dart`.

**Relevant KB:** `.ai-framework/knowledge-base/flutter/architecture/bloc-cubit-patterns.compact.md`

**Reference implementation:** `TicketsSettingsCubit` at `lib/features/ticket/shared/application/tickets_settings_cubit/tickets_settings_cubit.dart` — use its pending-request deduplication pattern.

## Before You Start

- [ ] Confirm task-02 is complete (`GetAppointmentTypesUseCase` exists)
- [ ] Read `lib/features/ticket/shared/application/tickets_settings_cubit/tickets_settings_cubit.dart` (pending-request pattern to copy)
- [ ] Read `lib/core/utils/cubit_extension.dart` (safeEmit)
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/core/application/appointment_types_cubit.dart` | create | Cubit + state (use `part` file pattern if preferred, or inline state) |
| `syncro-flutter/lib/features/appointments/core/application/appointment_types_state.dart` | create | State classes (part of cubit file) |

### Do NOT Modify

- Any repository or use case files — owned by task-02
- Any view or page files — owned by phase-3/task-04

## Implementation Steps

### Step 1: Create state file

`appointment_types_state.dart` (part of `appointment_types_cubit.dart`):

```dart
part of 'appointment_types_cubit.dart';

sealed class AppointmentTypesState extends Equatable {
  const AppointmentTypesState();
  @override
  List<Object?> get props => [];
}

final class AppointmentTypesLoading extends AppointmentTypesState {
  const AppointmentTypesLoading();
}

final class AppointmentTypesLoaded extends AppointmentTypesState {
  const AppointmentTypesLoaded(this.types);
  final List<AppointmentTypeOption> types;
  @override
  List<Object?> get props => [types];
}

final class AppointmentTypesError extends AppointmentTypesState {
  const AppointmentTypesError(this.message);
  final String message;
  @override
  List<Object?> get props => [message];
}
```

### Step 2: Create cubit

`appointment_types_cubit.dart`:

```dart
import 'package:equatable/equatable.dart';
import 'package:syncro/core/utils/cubit_extension.dart';
import 'package:syncro/core/utils/logger.dart';
import 'package:syncro/features/appointments/core/domain/get_appointment_types_usecase.dart';
import 'package:syncro/features/ticket/ticket_home/domain/appointment_type_option.dart';

part 'appointment_types_state.dart';

class AppointmentTypesCubit extends Cubit<AppointmentTypesState> {
  AppointmentTypesCubit(this._getAppointmentTypesUseCase)
      : super(const AppointmentTypesLoading()) {
    _load();
  }

  final GetAppointmentTypesUseCase _getAppointmentTypesUseCase;
  Future<void>? _pendingLoad;

  Future<List<AppointmentTypeOption>> load() async {
    if (state is AppointmentTypesLoaded) {
      return (state as AppointmentTypesLoaded).types;
    }
    if (_pendingLoad != null) {
      await _pendingLoad;
    }
    return state is AppointmentTypesLoaded
        ? (state as AppointmentTypesLoaded).types
        : [];
  }

  Future<void> _load() async {
    _pendingLoad = _fetchTypes();
    await _pendingLoad;
    _pendingLoad = null;
  }

  Future<void> _fetchTypes() async {
    final result = await _getAppointmentTypesUseCase.call();
    result.fold(
      (failure) {
        logger('AppointmentTypesCubit: failed to load — ${failure.message}',
            title: 'AppointmentTypesCubit');
        safeEmit(AppointmentTypesError(failure.message));
      },
      (response) => safeEmit(AppointmentTypesLoaded(response.appointmentTypes)),
    );
  }
}
```

### Step 3: Export

Add exports to the appointments barrel or to a new `appointments/core/application/` barrel if one exists. If there is no barrel for `core/application/`, just ensure imports work via the full path — no barrel creation needed.

## Testing

- [ ] `AppointmentTypesCubit` emits `AppointmentTypesLoading` then `AppointmentTypesLoaded` on success
- [ ] `AppointmentTypesCubit` emits `AppointmentTypesLoading` then `AppointmentTypesError` on failure
- [ ] `load()` returns cached list when state is already `AppointmentTypesLoaded` (no second fetch)
- [ ] `load()` awaits the pending request when called during loading
- [ ] Use `bloc_test` `blocTest<AppointmentTypesCubit, AppointmentTypesState>` pattern
- [ ] `flutter analyze` clean

## Documentation / KB Updates

- [ ] No new KB doc required — cubit follows the established `safeEmit` + sealed state pattern

## Completion Criteria

- [ ] `AppointmentTypesCubit` and state classes created
- [ ] Auto-fetches on construction; deduplicates concurrent calls
- [ ] `load()` returns cached types or awaits pending fetch
- [ ] Unit tests with `bloc_test` covering success and failure paths
- [ ] `flutter analyze` clean
