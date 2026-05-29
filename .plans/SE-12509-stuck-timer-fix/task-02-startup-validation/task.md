# Task 02 — Startup Timer Validation Against Backend

## JIRA

[SE-12509](https://syncrotech.atlassian.net/browse/SE-12509)

## Objective

When the app launches and `TimerEntryManager` finds a `RUNNING_TIMER_ENTRY` in SharedPreferences,
validate against the backend that the stored timer is still active. If the backend shows no active
timer for that ticket, call `TimerEntryManager().reset()` to clear the stale state.

**This task auto-resolves Derrick's stuck timer on the first launch after the app update** — no
manual action required from the user.

## Dependencies

None — task-02 is independent.

## File Ownership

**Modify:**
- `syncro-flutter/lib/features/ticket/timer_entry_widget/timer_entry_manager.dart`

**Create:**
- `syncro-flutter/lib/features/ticket/timer_entry_widget/infrastructure/validate_running_timer_use_case.dart`
- `syncro-flutter/test/features/ticket/timer_entry_widget/infrastructure/validate_running_timer_use_case_test.dart`

**Modify (startup integration):**
- Identify and modify the splash/auth flow where this validation should be called after the user
  is authenticated. Likely candidates:
  - `syncro-flutter/lib/features/splash/` — if there's a startup cubit
  - `syncro-flutter/lib/features/home/` — if home initializes after auth
  - `syncro-flutter/lib/app/dependency/` — if there's an app-level init sequence

**Do NOT modify (owned by task-01):**
- `syncro-flutter/lib/features/ticket/timer_entry_widget/application/timer_entry_cubit.dart`

---

## Implementation Steps

### Step 1 — Create `ValidateRunningTimerUseCase`

Create `syncro-flutter/lib/features/ticket/timer_entry_widget/infrastructure/validate_running_timer_use_case.dart`.

This use case:
1. Reads `AppSharedPreferences.getRunningTimerEntry()` — if null, returns early (nothing to validate)
2. Calls `GET /api/v1/tickets/:id` using the stored `runningTimerEntry.id` (the ticket ID)
3. Examines the response's timer entries for any entry with `end_time == null` (active timer)
4. If no active timer found on backend → calls `TimerEntryManager().reset()`
5. Returns `Either<Failure, bool>` — `Right(true)` if timer is valid, `Right(false)` if stale and cleared

Use the existing `AppRequests.getTicketsById` endpoint (`GET /api/v1/tickets/:id`).

Follow the use case pattern from `add_edit_timer_entry/` — inject `NetworkService` or reuse the
existing tickets repository interface. Check `TicketsRepository` for a `getTicketById` method.

```dart
class ValidateRunningTimerUseCase {
  final TicketsRepository ticketsRepository; // or directly via NetworkService

  ValidateRunningTimerUseCase({required this.ticketsRepository});

  Future<Either<Failure, bool>> call() async {
    final entry = AppSharedPreferences.getRunningTimerEntry();
    if (entry == null) return const Right(true); // nothing to validate

    final ticketId = entry.id;
    if (ticketId == null) {
      TimerEntryManager().reset(); // corrupt entry — clear it
      return const Right(false);
    }

    final result = await ticketsRepository.getTicketById(ticketId);
    return result.fold(
      (failure) => Left(failure), // network error — don't clear, try again next launch
      (ticket) {
        final hasActiveTimer = ticket.timerEntries
            ?.any((t) => t.endTime == null) ?? false;
        if (!hasActiveTimer) {
          TimerEntryManager().reset();
          return const Right(false);
        }
        return const Right(true);
      },
    );
  }
}
```

> **Note on network failure**: if the API call fails (no network), do NOT clear the timer state —
> the user may genuinely have an active timer and we shouldn't destroy it due to a transient error.
> Stale entries will be cleared on the next successful validation.

### Step 2 — Find the right startup hook

Explore the app's post-auth initialization flow to find the best place to call this use case.
After authentication and before the main home screen is rendered is ideal.

**Call the use case once per app launch**, not on every screen navigation.

### Step 3 — Wire up the use case call

In the identified startup location, call `ValidateRunningTimerUseCase().call()` as a fire-and-forget
(`unawaited(...)`) or awaited depending on whether you want to block the home screen render.
Prefer `unawaited` to avoid delaying the UI.

---

## Acceptance Criteria

- On app launch with a stale `RUNNING_TIMER_ENTRY` (ticket has no active timer on backend), the
  timer entry is cleared from SharedPreferences
- On app launch with a valid `RUNNING_TIMER_ENTRY` (ticket has an active timer on backend), the
  timer entry is preserved and the timer displays correctly
- On app launch with no `RUNNING_TIMER_ENTRY` in SP, no API call is made
- On network failure during validation, the timer entry is NOT cleared (preserve-on-error)
- After the fix is deployed, Derrick's stuck 3412-hour timer disappears on first launch
- `fvm flutter analyze` passes with no new warnings

## Testing

Create `test/features/ticket/timer_entry_widget/infrastructure/validate_running_timer_use_case_test.dart`.

**Test cases**:

1. `call() — no entry in SP — returns Right(true) without making API call`
2. `call() — SP has entry — ticket has no active timer — calls reset() and returns Right(false)`
3. `call() — SP has entry — ticket has active timer — does NOT call reset() and returns Right(true)`
4. `call() — SP has entry — API returns Left(Failure) — does NOT call reset() and returns Left(Failure)`
5. `call() — SP has entry with null id — calls reset() and returns Right(false)`

Run: `fvm flutter test test/features/ticket/timer_entry_widget/`

## Relevant KB

- `docs/kb-projects/syncro-flutter/technical/architecture/architecture-patterns.md`
