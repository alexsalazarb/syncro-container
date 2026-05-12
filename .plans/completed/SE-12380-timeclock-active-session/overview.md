# SE-12380 — Time Clock: Fix Active Session Detection

## Summary

The Time Clock screen shows the wrong UI state (Clock In) when an admin edits a historical timelog from the web, even though the technician has an active open session. Root cause: `GET /api/v1/timelogs/last` returns the most recently **modified** record (ordered by `updated_at`). Editing a historical record refreshes its `updated_at`, making it the "last" — and since it has `outAt`, the app incorrectly shows Clock In.

**Status**: complete  
**Last Updated**: 2026-05-12  
**Jira**: [SE-12380](https://syncrotech.atlassian.net/browse/SE-12380)  
**Severity**: Sev-3  
**Affected versions**: 1.3.1, 1.4.0 (iOS + Android)

## Root Cause

`TimeClockRepositoryImpl.getLatestTimeClock()` calls `GET /api/v1/timelogs/last?user_id={id}`.  
BE orders results by `updated_at` DESC. Editing any historical timelog via the web bumps its `updated_at`, moving it to the top. Since that record has `outAt != null`, `isClockInActive` returns `true` → Clock In button is shown despite an open session existing.

**Confirmed in**: `lib/features/time_clock/infrastructure/time_clock_repository_impl.dart:36-48`

## Fix Strategy (pure mobile — no BE changes required)

Replace the `GET /timelogs/last` call with `GET /timelogs` (the existing list endpoint).  
From the list, detect the active session by finding the first entry where `isClockedIn == true` (`inAt != null && outAt == null`).  
If no open session exists, fall back to the most recent entry ordered by `createdAt` DESC.

The `TimeClockResponse` interface (single `TimeClock?`) stays unchanged — no modifications to use case, cubit, or state.

## Scope

**Project**: syncro-flutter  
**Branch base**: `main`

## Tasks

| # | Task | Status | Branch |
|---|------|--------|--------|
| 01 | [Fix active session detection in repository](task-01-fix-active-session-detection/task.md) | complete | `feature/SE-12380` |
| 02 | [Regression tests for active session detection](task-02-regression-test/task.md) | complete | `feature/SE-12380` |

## Dependency DAG

```
task-01 ──► task-02
```

task-02 depends on task-01 (tests verify the fixed behaviour).

## Success Criteria

- [ ] Editing any historical timelog from the web does not change the Clock In/Out state in the mobile app after relaunch.
- [ ] A technician with an active session always sees Clock Out / Out for Lunch after relaunching the app, regardless of web edits to historical records.
- [ ] `isClockInActive` returns `false` (active session) when an open timelog exists alongside a more recently modified historical record.
- [ ] All existing tests pass.
- [ ] Two regression tests added and passing: open-session-wins and no-open-session-shows-clock-in.

## Files Touched

| File | Change |
|------|--------|
| `lib/features/time_clock/infrastructure/time_clock_repository_impl.dart` | Replace `/last` endpoint with list + open-session filter |
| `test/features/time_clock/application/latest_time_clock_cubit_test.dart` | New regression tests (+ generated mocks) |
