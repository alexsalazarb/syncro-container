# Investigation: SE-12739 — Appointment Time Changes Revert After OK

**Investigated**: 2026-06-11
**Confidence**: High (primary cause) / Confirmed (secondary cause)

---

## Timeline

| Date | Event |
|------|-------|
| ~Feb 5, 2026 | Commit `5207f5ee` (SCLP-809) — replaced inline `CustomTimePicker` with `showCustomTimePickerModal` |
| ~Apr 1, 2026 | Bug first reported (Syncro version 1.3.1+406, iOS) |
| Apr 3, 2026 | SE-12739 filed |
| Apr 13, 2026 | Alex Schomburg confirms: works in v1.1.21+384 ("Save Time" button), broken in v1.4.0+412 ("OK" button) |

---

## Origin Analysis (Step 2a)

### Regression commit: `5207f5ee` — SCLP-809

Before this commit, appointment time selection used an inline `CustomTimePicker` widget embedded in the widget tree. The widget showed a form with "Save Time" button. When "Save Time" was pressed, `onSave(_selectedTime.value)` was called synchronously — no async involved.

After this commit:
- `CustomTimePicker` (inline widget) was replaced with a modal via `showCustomTimePickerModal`
- `CustomDateTimePicker` became a `StatefulWidget`; `didUpdateWidget` now detects `mode = DateRadioButton.time` and calls `showCustomTimePickerModal` via `addPostFrameCallback`
- The time update path is now: `Future<TimeOfDay?>` → `.then((result) { onTimeChanged(result) })`
- Any condition that causes `result` to be `null` silently prevents the time from being updated

---

## Bug #1 — Primary: Wrong callback order in `.then()` handler

**File**: `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_datetime_picker.dart`

```dart
// BEFORE (wrong order):
showCustomTimePickerModal(...).then((result) {
  widget.onModeChange(null);     // ← reset fires FIRST
  if (result != null) {
    widget.onTimeChanged(result);  // ← time saved AFTER the reset
  }
});
```

`onModeChange(null)` fires first, resetting `startRadioButtonType` to null in the cubit. This triggers `startRadioButtonTypeRefresh()`, which after 500 ms re-emits `DateRadioButton.time` (Bug #4). At this point `widget.time` is still the **original** time because `onTimeChanged` hasn't been called yet. So `didUpdateWidget` opens a second modal with `initTime = widget.time` (old value), overwriting the user's selection.

**Fix**: Move `onModeChange(null)` AFTER the null check and `onTimeChanged` call, so the time is saved before the radio button state resets:

```dart
// AFTER (correct order):
showCustomTimePickerModal(...).then((result) {
  if (result != null) {
    widget.onTimeChanged(result);  // ← time saved FIRST
  }
  widget.onModeChange(null);     // ← reset fires AFTER
});
```

---

## Bug #2 — `MediaQuery` wrapper in wrong position in `showCustomTimePickerModal`

**File**: `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_time_picker.dart`

The `MediaQuery` override (`alwaysUse24HourFormat: false`) was placed inside `_buildTimePickerTheme` as its outermost wrapper. This caused it to be applied at the wrong tree level relative to Flutter's internal time picker widgets, breaking 12-hour format display in some device locales.

**Fix**: Move the `MediaQuery` wrapper out of `_buildTimePickerTheme` and into the `builder` callback directly, wrapping `_buildTimePickerTheme` as a child:

```dart
// AFTER:
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
```

Also removed the `async` keyword from `showCustomTimePickerModal` since `showTimePicker` already returns a `Future<TimeOfDay?>` directly — no `await` needed.

---

## Bug #3 — `CustomTimePickerDialog._handleOk` missing `form.save()`

**File**: `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/time_picker.dart` (~line 2735)

```dart
void _handleOk() {
  if (_entryMode.value == TimePickerEntryMode.input ||
      _entryMode.value == TimePickerEntryMode.inputOnly) {
    final FormState form = _formKey.currentState!;
    if (!form.validate()) { return; }
    // ❌ MISSING: form.save()
    widget.onSave.call(_selectedTime.value);
  }
  // ❌ For dial/dialOnly mode: widget.onSave is NEVER called
}
```

Flutter's standard `_handleOk` calls `form.save()` before reading `_selectedTime.value`. Without it, the `onSaved`/`onEditingComplete` callbacks on the text fields are never called, so `_selectedTime.value` may not reflect the user's latest typed input in some edge cases (e.g., field focused, value typed, OK tapped without removing focus first).

**Current status**: `CustomTimePickerDialog` is called only by `CustomTimePicker`, which is dead code since SCLP-809 — it's no longer used in any appointment flow. However, this is a correctness bug and should be fixed.

**Fix**: Add `form.save()` before `widget.onSave.call(...)`.

---

## Bug #4 — EDIT cubit re-emits `DateRadioButton.time` after 500 ms

**File**: `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart:208-258`

```dart
Future<void> startRadioButtonTypeRefresh() async {
  if (stateLoaded.startRadioButtonType == DateRadioButton.time) {
    safeEmit(stateLoaded.copyWith(removeStartButtonType: true, ...));
    await Future.delayed(const Duration(milliseconds: 500));
    // ❌ BUG: re-emits DateRadioButton.time — can re-trigger didUpdateWidget
    safeEmit(stateLoaded.copyWith(
      startRadioButtonType: DateRadioButton.time,
      startRadioButtonChanged: true,
    ));
  }
}
```

The Create cubit's equivalent method does NOT have the second `safeEmit` — it correctly terminates after the delay. The Edit cubit's version re-emits `DateRadioButton.time`, which causes `CustomDateTimePicker.didUpdateWidget` to detect a `null → DateRadioButton.time` transition and open a SECOND time picker modal. This second modal opens with `initTime = widget.time` (the time as-of the re-emit, which may be the original time if the first modal's update hasn't landed yet).

**Trigger condition**: `startRadioButtonTypeRefresh()` is called from `onEndTimeChanged` in the Edit view. It only runs its body if `startRadioButtonType == DateRadioButton.time` at call time. In normal single-picker usage this is null (the picker was already closed). However, in rapid-interaction scenarios (user quickly opens/closes pickers) the timing window can be hit.

**Fix**: Remove the second `safeEmit` from both `startRadioButtonTypeRefresh` and `endRadioButtonTypeRefresh` in the Edit cubit to match the Create cubit's behavior.

---

## Why Existing Tests Didn't Catch This

- Unit tests for `AppointmentCreateCubit` and `AppointmentEditCubit` test `changeStartTime`/`changeEndTime` in isolation — they don't exercise the `showCustomTimePickerModal` → `didUpdateWidget` → `.then()` chain.
- Widget tests for the appointment screens (if any) would use `showTimePicker` stubs and wouldn't exercise the real back-gesture dismissal path.
- The `PopScope` gap requires real device gesture testing to reproduce.

---

## Bug #5 — Double timezone conversion in `TimezoneService.convertToTimezone`

**File**: `syncro-flutter/lib/core/services/timezone_service.dart`

When a non-UTC `DateTime` was passed to `convertToTimezone`, the code converted it to UTC first with `.toUtc()` and then applied `tz.TZDateTime.from(utcDateTime, location)`. For datetimes that were already expressed as wall-clock time in the target timezone (naive, `isUtc == false`), this caused the timezone offset to be applied twice — shifting the time by the UTC offset amount.

This manifested when the user changed the **date** in the appointment form: the date change path went through `convertToTimezone` with a naive datetime, shifting the start/end time by the device's UTC offset (e.g., -5h → time moved 5 hours earlier).

**Fix**: When the datetime is not UTC, construct a `tz.TZDateTime` directly from its year/month/day/hour/minute components to avoid the double-conversion:

```dart
// BEFORE:
final utcDateTime = dateTime.toUtc();
return tz.TZDateTime.from(utcDateTime, location);

// AFTER:
return tz.TZDateTime(
  location,
  dateTime.year,
  dateTime.month,
  dateTime.day,
  dateTime.hour,
  dateTime.minute,
);
```

This was an unplanned hotfix discovered during manual validation of the primary fix.

---

## Files Modified by This Fix

| File | Change |
|------|--------|
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_datetime_picker.dart` | Swap callback order: `onTimeChanged` before `onModeChange(null)` |
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/custom_time_picker.dart` | Move `MediaQuery` wrapper into builder; remove `async` from function |
| `syncro-flutter/lib/core/global_widgets/custom_datetime_picker/time_picker.dart` | Fix `_handleOk`: add `form.save()` |
| `syncro-flutter/lib/features/appointments/appointment_edit/application/appointment_edit_cubit.dart` | Remove second `safeEmit` from `startRadioButtonTypeRefresh` and `endRadioButtonTypeRefresh` |
| `syncro-flutter/lib/core/services/timezone_service.dart` | Fix double timezone conversion for non-UTC datetimes |
