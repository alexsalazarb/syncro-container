# Task: Remove Re-emit in Edit Cubit Refresh Methods

**Plan**: SE-12739-appointment-time-revert
**Task ID**: task-02
**Task Path**: task-02-fix-edit-cubit-refresh
**Depends On**: None
**Ticket**: SE-12739

## Objective

Fix `AppointmentEditCubit.startRadioButtonTypeRefresh` and `endRadioButtonTypeRefresh` to NOT re-emit `DateRadioButton.time` after their 500 ms delay, aligning them with `AppointmentCreateCubit`'s correct implementation.

## Context

See [investigation.md](../investigation.md) Bug #4 for full details.

The Edit cubit's `startRadioButtonTypeRefresh` and `endRadioButtonTypeRefresh` contain a second `safeEmit` after a 500 ms `Future.delayed` that re-emits `startRadioButtonType: DateRadioButton.time` (or `endRadioButtonType: DateRadioButton.time`). This is incorrect — it causes `CustomDateTimePicker.didUpdateWidget` to detect a `null → DateRadioButton.time` mode transition and open a **second** time picker modal with `initTime = widget.time` (the current time, which may be the original value if the first modal's state update is still in flight).

The Create cubit's equivalent methods have NO second `safeEmit` — they terminate after the delay. The Edit cubit should match.

**File**: `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart`

Current (wrong) implementation of `startRadioButtonTypeRefresh` (lines 208–225):
```dart
Future<void> startRadioButtonTypeRefresh() async {
  if (stateLoaded.startRadioButtonType == DateRadioButton.time) {
    late AppointmentEditLoaded newState;
    newState = stateLoaded.copyWith(
      removeStartButtonType: true,
      startRadioButtonChanged: true,
    );
    safeEmit(newState);

    await Future.delayed(const Duration(milliseconds: 500));

    newState = stateLoaded.copyWith(           // ← BUG: re-emits DateRadioButton.time
      startRadioButtonType: DateRadioButton.time,
      startRadioButtonChanged: true,
    );
    safeEmit(newState);                        // ← BUG: triggers second modal
  }
}
```

Same pattern exists in `endRadioButtonTypeRefresh` (lines 241–258).

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read [investigation.md](../investigation.md) — specifically Bug #4
- [ ] Compare with `AppointmentCreateCubit.startRadioButtonTypeRefresh` to confirm the correct pattern
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` | modify | Remove second `safeEmit` from both refresh methods |

### Do NOT Modify

- `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart` — already correct, no changes needed
- Any custom_datetime_picker files — owned by task-01-fix-modal-time-capture

## Implementation Steps

### Step 1: Fix `startRadioButtonTypeRefresh`

Remove the `await Future.delayed` block's second `safeEmit`. The method should only emit the "remove" state and then terminate:

```dart
// After (matches create cubit pattern):
Future<void> startRadioButtonTypeRefresh() async {
  if (stateLoaded.startRadioButtonType == DateRadioButton.time) {
    safeEmit(stateLoaded.copyWith(
      removeStartButtonType: true,
      startRadioButtonChanged: true,
    ));
    await Future.delayed(const Duration(milliseconds: 500));
    // No second emit — intentionally removed (was re-triggering time picker modal)
  }
}
```

### Step 2: Fix `endRadioButtonTypeRefresh`

Apply the same fix to `endRadioButtonTypeRefresh`:

```dart
// After (matches create cubit pattern):
Future<void> endRadioButtonTypeRefresh() async {
  if (stateLoaded.endRadioButtonType == DateRadioButton.time) {
    safeEmit(stateLoaded.copyWith(
      removeEndButtonType: true,
      endRadioButtonChanged: true,
    ));
    await Future.delayed(const Duration(milliseconds: 500));
    // No second emit — intentionally removed (was re-triggering time picker modal)
  }
}
```

### Step 3: Verify Create cubit is unchanged

Confirm `AppointmentCreateCubit.startRadioButtonTypeRefresh` and `endRadioButtonTypeRefresh` have no second `safeEmit` (they should be correct already — verify, don't modify).

## Testing

- [ ] Run `fvm flutter test` in `syncro-flutter/` — existing tests pass
- [ ] Run `fvm flutter analyze` — no new warnings
- [ ] Run `pre-commit-check` before staging

## Documentation / KB Updates

- [ ] No KB/doc updates required

## Completion Criteria

- [ ] `startRadioButtonTypeRefresh` in Edit cubit no longer re-emits `DateRadioButton.time`
- [ ] `endRadioButtonTypeRefresh` in Edit cubit no longer re-emits `DateRadioButton.time`
- [ ] `fvm flutter test` passes
- [ ] `fvm flutter analyze` passes
- [ ] Changes committed to `plan/SE-12739-appointment-time-revert/task-02-fix-edit-cubit-refresh` branch
- [ ] Status updated in `status.md`
