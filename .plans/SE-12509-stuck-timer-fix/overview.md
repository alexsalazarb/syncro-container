# Plan: SE-12509 — Stuck Ticket Timer Fix

## Objective

Fix a stuck ticket timer that displays a 3412-hour running timer across all tickets for a specific
user and persists across app reinstalls. Root cause: stale `RUNNING_TIMER_ENTRY` in SharedPreferences
surviving Android Auto-Backup + a `reset()` call that's missing in the stop-failure path.

## Type

Type 3 — Bug/Support (reactive fix for production issue)

## JIRA

[SE-12509](https://syncrotech.atlassian.net/browse/SE-12509)

## Master Plan

None

## Project

syncro-flutter

## Branch Convention

`plan/SE-12509-stuck-timer-fix/{task-path}`

## Demo Date

TBD

## Dev / QA

- **Dev**: Alex Salazar
- **QA**: Unassigned

---

## Context

A single technician (Derrick Finney, user_id: 152250) sees a 3412-hour stuck timer displayed on
every ticket in the mobile app. The timer persists after clearing app cache and reinstalling.

**Root cause** — two compounding issues:

1. **Missing `reset()` on stop failure** (`timer_entry_cubit.dart:89-91`): `handleStop()` only calls
   `timerEntryManager.reset()` on API success. On failure, SharedPreferences retains the
   `RUNNING_TIMER_ENTRY` entry. On next app launch, `TimerEntryManager` reads it, sees a non-null
   `startTime`, and sets `_isTimerRunning = true` → stuck timer appears.

2. **Android Auto-Backup preserves stale SP data across reinstalls**: SharedPreferences is included
   in Android's automatic cloud backup by default (Android 6.0+). When the user uninstalls and
   reinstalls, the stale `RUNNING_TIMER_ENTRY` is restored from Google Drive backup. This is why
   reinstall alone does not fix the issue.

**Immediate fix for Derrick (no deploy needed)**: Go to
`Settings > Apps > Syncro > Storage > Clear Data` (not just Clear Cache).

**Permanent fix via this plan**: task-02 (startup validation) auto-resolves Derrick's stuck timer
on the next app update without any manual steps from the user.

---

## Architecture

```
App launch
  │
  ├─ TimerEntryManager._internal()
  │     reads RUNNING_TIMER_ENTRY from SharedPreferences
  │     if startTime != null → _isTimerRunning = true  ← stale state lives here
  │
  ├─ [task-02] ValidateRunningTimerUseCase
  │     called during splash/auth flow (after user is known)
  │     GET /api/v1/tickets/:id (stored ticketId)
  │     if no active timer on backend → TimerEntryManager().reset()
  │
  └─ TimerEntryCubit.onInit()
        if isTimerRunning → handleStart()  ← would NOT fire after task-02 clears state

Stop flow (user taps Stop):
  TimerEntryCubit.handleStop()
    │
    ├─ timerEntryManager.stopTimer()      ← clears in-memory only
    ├─ POST /api/v1/tickets/:id/timer_entry
    ├─ on SUCCESS → timerEntryManager.reset()   ← clears SharedPreferences ✅
    └─ [task-01] on FAILURE → timerEntryManager.reset()  ← NEW: clears SP even on error ✅
```

---

## Dependency DAG

```
task-01-stop-failure-reset      (independent)
task-02-startup-validation      (independent)
task-03-android-backup          (independent)
```

All 3 tasks are fully independent and can run in parallel.

---

## Kill Criteria

1. Backend `GET /api/v1/tickets/:id` does not return timer entry data that lets us determine
   whether a timer is currently active — escalate to backend team for a dedicated endpoint.
2. Android backup rules XML format is rejected at runtime (API level incompatibility) — investigate
   `android:dataExtractionRules` (API 31+) as an alternative to `fullBackupContent`.

---

## Tasks

| ID | Slug | Status | Owner Files |
|----|------|--------|-------------|
| task-01 | stop-failure-reset | not-started | `timer_entry_cubit.dart` |
| task-02 | startup-validation | not-started | `timer_entry_manager.dart`, new use case, splash/auth startup |
| task-03 | android-backup | not-started | `AndroidManifest.xml`, new `backup_rules.xml` |

---

## Relevant KB

- `docs/kb-projects/syncro-flutter/technical/architecture/architecture-patterns.md` — Cubit patterns, safeEmit
