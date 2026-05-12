# Investigation — SE-12380 Time Clock Active Session Detection

## Trigger

Customer bug report (03/20/2026). Reproduced internally on Android (SA 80040) and iOS (SA 61524). Confirmed still present in v1.4.0.

## Origin Analysis

Feature introduced in commit `b18808b9` (`SCLP-660: Mobile App: Time Clock List`, 2025-10-15).  
The `GET /timelogs/last` endpoint and `isClockInActive` logic were both introduced in that commit.  
No subsequent commit modified the active-session detection strategy.

## Root Cause

### Entry point

`TimeClockRepositoryImpl.getLatestTimeClock()` — `lib/features/time_clock/infrastructure/time_clock_repository_impl.dart:36`

```dart
return networkService.sendRequestCall<TimeClockResponse>(
  request: AppRequests.getLatestTimeClock.requestOption(), // → GET /api/v1/timelogs/last
  deserializer: LatestTimeClockDeserializer(),
  queryParameters: {'user_id': params.userId.toString()},
);
```

### BE behaviour

`GET /api/v1/timelogs/last` returns the timelog with the highest `updated_at`. When a historical (closed) timelog is edited from the web, its `updated_at` is refreshed and it becomes "the last" — even though an open session (`outAt == null`) exists.

### State detection

`LatestTimeClockCubit.isClockInActive` — `lib/features/time_clock/application/latest_time_clock_cubit.dart:67`

```dart
bool get isClockInActive {
  final timeClock = state is LatestTimeClockLoaded
      ? (state as LatestTimeClockLoaded).timeClock
      : null;
  if (timeClock == null) return true;        // no data → Clock In
  else if (timeClock.outAt != null) return true; // has outAt → Clock In  ← BUG MANIFESTS HERE
  return false;
}
```

The logic is correct in isolation — but it operates on the **wrong record**. The historical record returned by `/timelogs/last` has `outAt != null`, so the getter shows Clock In.

### Why existing tests missed this

No tests existed for `LatestTimeClockCubit` state detection. The feature was delivered without cubit-level tests.

## Existing assets

- `GET /api/v1/timelogs?user_id={id}` already implemented in `TimeClockRepositoryImpl.getTimeClockList()` — returns all timelogs for the user.
- `TimeClock.isClockedIn` domain getter already correctly implements `inAt != null && outAt == null`.
- `TimeClockResponse` (wraps a single `TimeClock?`) — interface stays unchanged.

## Fix confidence: HIGH

The fix is scoped to a single method in the repository implementation. All upstream layers (use case, cubit, state, UI) remain unchanged. The existing list endpoint is already wired up and tested in production via `TimeClockDetailCubit`.

## What the fix does NOT address

Real-time sync while the screen is open (Scenario 2 — separate ticket/task). This plan covers only Scenario 1: wrong state on app relaunch after web edit of historical timelog.
