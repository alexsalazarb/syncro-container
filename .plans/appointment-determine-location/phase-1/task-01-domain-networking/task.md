# Task 01: Domain & Networking

**Plan**: appointment-determine-location
**Phase**: 1 — Foundation
**Status**: not-started
**Depends on**: —

## Objective

Create all domain models, the new networking endpoint, the repository method, and the use case for the `GET /appointment_types/:id/determine_location` call.

## Steps

1. **Add endpoint** in `app_requests_appointments.dart`:
   ```dart
   static String determineAppointmentLocation(int appointmentTypeId) =>
       '${AppLinks.baseAppLink}appointment_types/$appointmentTypeId/determine_location';
   ```
   Query params (`customer_id`, `ticket_id`) are optional — passed at call time, not baked into the URL.

2. **Create `DetermineAppointmentLocationParams`** in `lib/features/appointments/core/domain/`:
   ```dart
   class DetermineAppointmentLocationParams {
     final int appointmentTypeId;
     final int? customerId;
     final int? ticketId;
   }
   ```

3. **Create `DetermineAppointmentLocationResponse`** in the same directory:
   - Parses `{ "appointment_location": "US" }`
   - Field: `appointmentLocation: String`
   - Follow existing `fromJson` patterns in the codebase

4. **Add `determineLocation()` to repository interface** (`appointments_repository.dart`):
   ```dart
   Future<Either<Failure, DetermineAppointmentLocationResponse>> determineLocation(
     DetermineAppointmentLocationParams params,
   );
   ```

5. **Implement in `appointments_repository_impl.dart`**:
   - Use `NetworkService.sendRequestCall()` (same pattern as `getAppointmentTypes`)
   - Build query map from params: include `customer_id` only if `params.customerId != null`, same for `ticket_id`
   - Deserialize with `DetermineAppointmentLocationResponse.fromJson`

6. **Create `DetermineAppointmentLocationUseCase`** in `lib/features/appointments/core/domain/use_cases/`:
   - Follows the existing use case pattern in that directory
   - Input: `DetermineAppointmentLocationParams`
   - Output: `Either<Failure, DetermineAppointmentLocationResponse>`

## File Ownership

- `syncro-flutter/lib/core/networking/commons/app_requests_appointments.dart`
- `syncro-flutter/lib/features/appointments/core/domain/determine_appointment_location_params.dart` (new)
- `syncro-flutter/lib/features/appointments/core/domain/determine_appointment_location_response.dart` (new)
- `syncro-flutter/lib/features/appointments/core/domain/appointments_repository.dart`
- `syncro-flutter/lib/features/appointments/core/infrastructure/appointments_repository_impl.dart`
- `syncro-flutter/lib/features/appointments/core/domain/use_cases/determine_appointment_location_use_case.dart` (new)

## Success Criteria

- [ ] `AppRequests.determineAppointmentLocation(id)` returns the correct URL
- [ ] `DetermineAppointmentLocationParams` accepts `appointmentTypeId` (required) and `customerId`/`ticketId` (optional)
- [ ] `DetermineAppointmentLocationResponse.fromJson` correctly parses `{ "appointment_location": "US" }`
- [ ] `AppointmentsRepository` interface declares `determineLocation()`
- [ ] `AppointmentsRepositoryImpl` calls the correct endpoint with optional query params (omits nulls)
- [ ] `DetermineAppointmentLocationUseCase` compiles and follows existing use case patterns
- [ ] `flutter analyze` passes with no new warnings
