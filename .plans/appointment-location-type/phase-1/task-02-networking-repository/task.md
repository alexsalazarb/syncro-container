# Task: Networking & Repository

**Plan**: appointment-location-type
**Phase**: 1
**Task ID**: task-02
**Task Path**: phase-1/task-02-networking-repository
**Depends On**: phase-1/task-01-domain-models
**JIRA**: SE-11616

## Objective

Add the `GET /api/v1/appointment_types` endpoint to the networking layer, create the response model and deserializer, add `getAppointmentTypes()` to the repository interface and implementation, and expose a use case.

## Context

**Endpoint:** `GET /api/v1/appointment_types`
**Confirmed response shape** (verified against actual BE response):
```json
{
  "appointment_types": [
    { "id": 549, "name": "Onsite",          "location_type": "customer",   "location_hard_code": null,                          "account_id": 224, ... },
    { "id": 548, "name": "Our Office",       "location_type": "shop",       "location_hard_code": null,                          "account_id": 224, ... },
    { "id": 550, "name": "Phone Call",       "location_type": "pre_defined","location_hard_code": "Phone Call",                  "account_id": 224, ... },
    { "id": 551, "name": "Remote Support",   "location_type": "pre_defined","location_hard_code": "Remote Support Link: (edit me)","account_id": 224, ... }
  ]
}
```
Root key is `appointment_types`. Each item has `id` (int), `name` (String), `location_type` (String), `location_hard_code` (String?). Additional fields (`account_id`, `email_instructions`, `buffer`, `appointment_reminders_schedule_id`, `product_id`, `created_at`, `updated_at`) are present but **ignored** by the deserializer — only parse the four fields above.

**Existing patterns to follow:**
- `AppRequests` enum is in `lib/core/networking/commons/app_requests.dart`
- Appointment-specific endpoints live in `lib/core/networking/commons/app_requests_appointments.dart` (part file)
- Repository interface: `lib/features/appointments/core/domain/appointments_repository.dart`
- Repository impl: `lib/features/appointments/core/infrastructure/appointments_repository_impl.dart`
- Use case base class: `lib/core/usecases/usecase.dart` — extend `UseCaseNoParams<T>` (or similar) since this call takes no params
- Deserializer pattern: look at `get_appointment_by_id_deserializer.dart` and `appointments_deserializer.dart` for reference
- `AppointmentTypeOption.fromJson()` already exists (updated in task-01 to include `locationType`) — reuse it in the deserializer

**Relevant KB:** `.ai-framework/knowledge-base/flutter/architecture/functional-error-handling.compact.md` — Either/Failure patterns for use cases and repositories.

## Before You Start

- [ ] Confirm task-01 is complete (AppointmentTypeOption.fromJson includes locationType)
- [ ] Read `lib/core/networking/commons/app_requests.dart` (enum values and structure)
- [ ] Read `lib/core/networking/commons/app_requests_appointments.dart` (part file pattern)
- [ ] Read `lib/features/appointments/core/infrastructure/get_appointment_by_id_deserializer.dart` (deserializer pattern)
- [ ] Read `lib/core/usecases/usecase.dart` (base class to extend)
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/networking/commons/app_requests.dart` | modify | Add `getAppointmentTypes` to enum |
| `syncro-flutter/lib/core/networking/commons/app_requests_appointments.dart` | modify | Add `getAppointmentTypes` case in `_appointmentsRequestOption()` |
| `syncro-flutter/lib/features/appointments/core/domain/get_appointment_types_response.dart` | create | Response model wrapping `List<AppointmentTypeOption>` |
| `syncro-flutter/lib/features/appointments/core/domain/get_appointment_types_usecase.dart` | create | Use case with no params |
| `syncro-flutter/lib/features/appointments/core/domain/appointments_repository.dart` | modify | Add `getAppointmentTypes()` method signature |
| `syncro-flutter/lib/features/appointments/core/infrastructure/appointment_types_deserializer.dart` | create | Parses list of AppointmentTypeOption from response |
| `syncro-flutter/lib/features/appointments/core/infrastructure/appointments_repository_impl.dart` | modify | Implement `getAppointmentTypes()` |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/domain/appointment_type_option.dart` — owned by phase-1/task-01-domain-models
- `syncro-flutter/lib/features/appointments/core/domain/appointment_location_type.dart` — owned by phase-1/task-01-domain-models
- Any cubit or view files — owned by later tasks

## Implementation Steps

### Step 1: Add enum value and endpoint

In `app_requests.dart`, add `getAppointmentTypes` to the enum (near the other appointment values).

In `app_requests_appointments.dart`, add a case inside `_appointmentsRequestOption()`:
```dart
AppRequests.getAppointmentTypes => RequestOption(
  requestType: RequestType.rest,
  requestOperation: RestRequestType.get,
  requestName: AppRequests.getAppointmentTypes.name,
  operationNameOrPath: "$_apiVersion/appointment_types",
  documentNode: "",
),
```

### Step 2: Create response model

`get_appointment_types_response.dart`:
```dart
import 'package:equatable/equatable.dart';
import 'package:syncro/features/ticket/ticket_home/domain/appointment_type_option.dart';

class GetAppointmentTypesResponse extends Equatable {
  const GetAppointmentTypesResponse({required this.appointmentTypes});
  final List<AppointmentTypeOption> appointmentTypes;

  @override
  List<Object?> get props => [appointmentTypes];
}
```

### Step 3: Create deserializer

`appointment_types_deserializer.dart` — parse the `appointment_types` array from the JSON root. `AppointmentTypeOption.fromJson` (updated in task-01) already handles `id`, `name`, `location_type`, and `location_hard_code` — no extra mapping needed here:
```dart
class AppointmentTypesDeserializer
    implements Deserializer<GetAppointmentTypesResponse> {
  @override
  GetAppointmentTypesResponse fromMap(Map<String, dynamic> map) {
    final rawList = map['appointment_types'] as List<dynamic>? ?? [];
    final types = rawList
        .whereType<Map<String, dynamic>>()
        .map(AppointmentTypeOption.fromJson)
        .toList();
    return GetAppointmentTypesResponse(appointmentTypes: types);
  }
}
```

Check `Deserializer` import path from existing deserializers in the same `infrastructure/` folder.

### Step 4: Update repository interface

Add to `AppointmentsRepository`:
```dart
Future<Either<Failure, GetAppointmentTypesResponse>> getAppointmentTypes();
```

Import `get_appointment_types_response.dart`.

### Step 5: Implement in repository

In `AppointmentsRepositoryImpl`:
```dart
@override
Future<Either<Failure, GetAppointmentTypesResponse>> getAppointmentTypes() async {
  try {
    return networkService.sendRequestCall(
      request: AppRequests.getAppointmentTypes.requestOption(),
      deserializer: AppointmentTypesDeserializer(),
    );
  } catch (e) {
    return Left(UnexpectedFailure(e.toString()));
  }
}
```

### Step 6: Create use case

`get_appointment_types_usecase.dart`:
```dart
class GetAppointmentTypesUseCase {
  GetAppointmentTypesUseCase(this._repository);
  final AppointmentsRepository _repository;

  Future<Either<Failure, GetAppointmentTypesResponse>> call() =>
      _repository.getAppointmentTypes();
}
```

(Check if the project has a `UseCaseNoParams` base class to extend instead of a plain class.)

### Step 7: Export from barrel

Add exports to `lib/features/appointments/appointments.dart` (or the relevant barrel file):
- `get_appointment_types_response.dart`
- `get_appointment_types_usecase.dart`

## Testing

- [ ] `AppointmentTypesDeserializer.fromMap()` correctly parses a list with mixed `location_type` values including `null`
- [ ] `AppointmentTypesDeserializer.fromMap()` returns empty list when `appointment_types` is absent or null
- [ ] `GetAppointmentTypesUseCase.call()` returns `Right(GetAppointmentTypesResponse)` on success
- [ ] `GetAppointmentTypesUseCase.call()` returns `Left(Failure)` on network error
- [ ] Mock `AppointmentsRepository` in existing tests still compiles (it now has a new method — add mock stub)
- [ ] `flutter analyze` clean

## Documentation / KB Updates

- [ ] No new KB doc needed — follows existing repository and use-case patterns already documented

## Completion Criteria

- [ ] `getAppointmentTypes` enum value present in `AppRequests`
- [ ] `/api/v1/appointment_types` endpoint registered in `app_requests_appointments.dart`
- [ ] `GetAppointmentTypesResponse` model and `AppointmentTypesDeserializer` created
- [ ] `AppointmentsRepository` interface includes `getAppointmentTypes()`
- [ ] `AppointmentsRepositoryImpl` implements `getAppointmentTypes()` with standard error handling
- [ ] `GetAppointmentTypesUseCase` created and exported
- [ ] All tests pass; mock stubs updated where needed
