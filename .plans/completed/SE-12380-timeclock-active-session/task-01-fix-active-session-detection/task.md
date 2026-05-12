# Task 01 — Fix Active Session Detection in Repository

## Objective

Replace `GET /api/v1/timelogs/last` with the list endpoint in `TimeClockRepositoryImpl.getLatestTimeClock()`. Detect the active session by scanning the list for the first entry where `isClockedIn == true`. Fall back to the most recent entry by `createdAt` DESC if no open session exists.

## Context

- **File**: `lib/features/time_clock/infrastructure/time_clock_repository_impl.dart`
- **Method**: `getLatestTimeClock()` (line 36)
- **Current endpoint**: `GET /api/v1/timelogs/last` — returns most recently MODIFIED timelog (wrong)
- **Target endpoint**: `GET /api/v1/timelogs` — returns full list; filter client-side
- **Domain getter**: `TimeClock.isClockedIn` (`inAt != null && outAt == null`) already correct — use it
- **Interface unchanged**: `TimeClockResponse` still wraps a single `TimeClock?` — no callers change

## Implementation Steps

1. In `TimeClockRepositoryImpl.getLatestTimeClock()`:
   - Replace the `AppRequests.getLatestTimeClock` call with `AppRequests.getTimeClockList`
   - Use `TimeClockListDeserializer` (already available) to deserialize the list response
   - From the resulting `List<TimeClock>`, find the first entry where `timeClock.isClockedIn == true`
   - If found → that is the active session; wrap it in `TimeClockResponse(timeClock: activeEntry)`
   - If not found → sort by `createdAt` DESC, take the first; wrap in `TimeClockResponse(timeClock: mostRecent)` (or `TimeClockResponse(timeClock: null)` if the list is empty)

2. The existing `AppRequests.getLatestTimeClock` enum case and its `RequestOption` (`/timelogs/last`) can be left in place — they are no longer called but removing them is not in scope.

3. Run `fvm flutter analyze` to confirm no type errors.

## File Ownership

**MUST modify:**
- `lib/features/time_clock/infrastructure/time_clock_repository_impl.dart`

**MUST NOT modify:**
- `lib/features/time_clock/domain/get_latest_time_clock_usecase.dart`
- `lib/features/time_clock/application/latest_time_clock_cubit.dart`
- `lib/features/time_clock/application/latest_time_clock_state.dart`
- `lib/features/time_clock/domain/time_clock.dart`
- `lib/features/time_clock/domain/time_clock_response.dart`
- Any presentation or routing file

## Verification

- `fvm flutter analyze` — no errors
- `fvm flutter test lib/features/time_clock/` — all tests pass (task-02 will add new ones)
- Manual trace: the method now calls the list endpoint, scans for `isClockedIn == true`, and returns it wrapped in `TimeClockResponse`

## Acceptance Criteria

- [ ] `getLatestTimeClock()` no longer calls `GET /api/v1/timelogs/last`
- [ ] It calls `GET /api/v1/timelogs` and filters client-side
- [ ] When a timelog with `outAt == null` exists in the list, it is returned as the active session
- [ ] When no open timelog exists, the most recent by `createdAt` is returned (or null for empty list)
- [ ] `fvm flutter analyze` passes clean
