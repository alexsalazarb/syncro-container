# Task 01 — Fix Stop Failure: Call reset() Even on API Error

## JIRA

[SE-12509](https://syncrotech.atlassian.net/browse/SE-12509)

## Objective

In `TimerEntryCubit.handleStop()`, the call to `timerEntryManager.reset()` only happens on API
success. When `addTimerEntryUseCase` returns a `Left(Failure)`, SharedPreferences retains the
`RUNNING_TIMER_ENTRY` key. On the next app launch, `TimerEntryManager` reads it and reconstructs
a running timer — stuck timer appears.

Fix: also call `timerEntryManager.reset()` in the failure branch so SharedPreferences is always
cleared when the user intends to stop the timer.

> **Trade-off**: the unsaved timer entry data is lost when the API fails. This is acceptable because
> the current UX already loses the data silently (stop appears to work, timer reappears on restart).
> Clearing state is strictly better than leaving the user stuck.

## Dependencies

None — task-01 is independent.

## File Ownership

**Modify:**
- `syncro-flutter/lib/features/ticket/timer_entry_widget/application/timer_entry_cubit.dart`

**Create:**
- `syncro-flutter/test/features/ticket/timer_entry_widget/application/timer_entry_cubit_test.dart`

**Do NOT modify (owned by other tasks):**
- `syncro-flutter/lib/features/ticket/timer_entry_widget/timer_entry_manager.dart` — owned by task-02

---

## Implementation Steps

### Step 1 — Add `reset()` call in the failure branch of `handleStop()`

In `timer_entry_cubit.dart`, `handleStop()` currently (lines 88–91):

```dart
final result = await addTimerEntryUseCase.call(params);
return result.fold(
  (failure) {
    return false;  // ← SharedPreferences never cleared
  },
  (response) {
    timerEntryManager.reset();  // ← only called on success
    ...
  },
);
```

Change the failure branch to:

```dart
return result.fold(
  (failure) {
    timerEntryManager.reset();  // ← ADD: clear SP even on error
    return false;
  },
  (response) {
    timerEntryManager.reset();
    ...
  },
);
```

No other changes needed in this file.

---

## Acceptance Criteria

- When `addTimerEntryUseCase.call()` returns `Left(Failure)`, `timerEntryManager.reset()` is called
- When `addTimerEntryUseCase.call()` returns `Right(response)`, behavior is unchanged
- After a failed stop, relaunching the app does NOT show a stuck timer
- `fvm flutter analyze` passes with no new warnings

## Testing

Create `test/features/ticket/timer_entry_widget/application/timer_entry_cubit_test.dart`.

**Test cases** (use `bloc_test` + `mockito`):

1. `handleStop() — on API success — calls reset() and returns true`
2. `handleStop() — on API failure — calls reset() and returns false`
   - Mock `addTimerEntryUseCase` to return `Left(Failure('network error'))`
   - Verify `timerEntryManager.reset()` was called (spy or verify via state)
   - Verify return value is `false`

Reference existing cubit test patterns in `test/features/` for mock setup conventions.

Run: `fvm flutter test test/features/ticket/timer_entry_widget/application/`

## Relevant KB

- `docs/kb-projects/syncro-flutter/technical/architecture/architecture-patterns.md`
