# Task: Fix Time Modal Capture and Dead-Code form.save()

**Plan**: SE-12739-appointment-time-revert
**Task ID**: task-01
**Task Path**: task-01-fix-modal-time-capture
**Depends On**: None
**Ticket**: SE-12739

## Objective

Fix three related defects in the custom datetime picker: (1) swap callback order in `_CustomDateTimePickerState.didUpdateWidget`'s `.then()` — call `onTimeChanged` before `onModeChange(null)`; (2) move the `MediaQuery` wrapper inside `showCustomTimePickerModal`'s builder and remove its `async`; (3) fix `CustomTimePickerDialog._handleOk` to call `form.save()` before reading `_selectedTime.value`.

## Context

See [investigation.md](../investigation.md) for full root cause details.

The appointment time picker was broken by SCLP-809 (commit `5207f5ee`) which replaced an inline `CustomTimePicker` with a modal `showCustomTimePickerModal`. The root cause was the callback order in the `.then()` handler: `onModeChange(null)` fired before `onTimeChanged`, causing the cubit's radio button state to reset before the time was saved — which triggered a second modal opening with the original time.

**File locations**:
- `.then()` callback: `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_datetime_picker.dart` (inside `didUpdateWidget`, after the `addPostFrameCallback`)
- `showCustomTimePickerModal` builder: `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_time_picker.dart`
- `_handleOk`: `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/time_picker.dart` (search for `void _handleOk()` inside `_TimePickerDialogState`)

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read the ticket SE-12739: check description AND comments
- [ ] Read [investigation.md](../investigation.md) for full root cause context
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_datetime_picker.dart` | modify | Swap callback order: `onTimeChanged` before `onModeChange(null)` |
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_time_picker.dart` | modify | Move `MediaQuery` into builder; remove `async` |
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/time_picker.dart` | modify | Add `form.save()` in `_handleOk` |

### Do NOT Modify

- `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` — owned by task-02-fix-edit-cubit-refresh
- `syncro-flutter/lib/features/appointments/appointment_create/application/appointment_create_cubit.dart` — not changed (already correct)

## Implementation Steps

### Step 1: Fix callback order in `.then()` handler

In `custom_datetime_picker.dart`, inside `didUpdateWidget`, swap the order so `onTimeChanged` fires before `onModeChange(null)`:

```dart
// Before (wrong — resets state before saving time):
showCustomTimePickerModal(...).then((result) {
  widget.onModeChange(null);
  if (result != null) {
    widget.onTimeChanged(result);
  }
});

// After (correct — time saved first, then state reset):
showCustomTimePickerModal(...).then((result) {
  if (result != null) {
    widget.onTimeChanged(result);
  }
  widget.onModeChange(null);
});
```

### Step 2: Move `MediaQuery` wrapper inside `showCustomTimePickerModal` builder

In `custom_time_picker.dart`, remove the `MediaQuery` wrapper from inside `_buildTimePickerTheme` and place it in the `builder` callback wrapping `_buildTimePickerTheme`. Also remove `async` from the function signature:

```dart
// After:
Future<TimeOfDay?> showCustomTimePickerModal(...) {  // no async
  return showTimePicker(
    ...
    builder: (BuildContext context, Widget? child) {
      return MediaQuery(
        data: MediaQuery.of(context).copyWith(alwaysUse24HourFormat: false),
        child: _buildTimePickerTheme(
          context: context,
          hourMinuteColor: hourMinuteColor,
          child: child!,
        ),
      );
    },
  );
}
```

### Step 3: Fix `CustomTimePickerDialog._handleOk` — add `form.save()`

In `time_picker.dart`, find `void _handleOk()` inside `_TimePickerDialogState` and add `form.save()` before reading `_selectedTime.value`:

```dart
// Before:
void _handleOk() {
  if (_entryMode.value == TimePickerEntryMode.input ||
      _entryMode.value == TimePickerEntryMode.inputOnly) {
    final FormState form = _formKey.currentState!;
    if (!form.validate()) {
      setState(() { _autoValidateMode.value = AutovalidateMode.always; });
      return;
    }
    widget.onSave.call(_selectedTime.value);
  }
}

// After:
void _handleOk() {
  if (_entryMode.value == TimePickerEntryMode.input ||
      _entryMode.value == TimePickerEntryMode.inputOnly) {
    final FormState form = _formKey.currentState!;
    if (!form.validate()) {
      setState(() { _autoValidateMode.value = AutovalidateMode.always; });
      return;
    }
    form.save();
    widget.onSave.call(_selectedTime.value);
  }
}
```

Note: `CustomTimePickerDialog` is currently dead code (not used in the appointment flow since SCLP-809). This fix is a correctness improvement.

## Testing

- [ ] Run `fvm flutter analyze` — no new warnings
- [ ] Run `fvm flutter test` in `syncro-flutter/` — existing tests pass
- [ ] Run `pre-commit-check` before staging
- [ ] Manual smoke test (if device available): Create appointment, change start/end time, confirm time saves

## Documentation / KB Updates

- [ ] No KB/doc updates required — the fix is self-contained in existing files

## Completion Criteria

- [x] Callback order fixed: `onTimeChanged` called before `onModeChange(null)` in `.then()`
- [x] `MediaQuery` wrapper moved inside builder; `async` removed from `showCustomTimePickerModal`
- [x] `form.save()` added before `widget.onSave.call(...)` in `_handleOk`
- [x] `fvm flutter analyze` passes
- [x] `fvm flutter test` passes
- [x] Changes committed — commit `eaa2b94e`
- [x] Status updated in `status.md`
