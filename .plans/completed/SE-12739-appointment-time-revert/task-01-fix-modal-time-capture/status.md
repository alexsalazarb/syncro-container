# Status: Fix Time Modal Capture and Dead-Code form.save()

**Current Status**: complete
**Last Updated**: 2026-06-11
**Agent**: —
**Branch**: plan/SE-12739-appointment-time-revert/task-01-fix-modal-time-capture
**PR**: —

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-06-11 | not-started | Task created |
| 2026-06-11 | complete | Callback order fix + MediaQuery placement + form.save() applied (commit eaa2b94e) |

## Blockers

None

## Artifacts

- `lib/core/global_widgets/custom_datetime_picker/custom_datetime_picker.dart` — swapped callback order
- `lib/core/global_widgets/custom_datetime_picker/custom_time_picker.dart` — MediaQuery placement + removed async
- `lib/core/global_widgets/custom_datetime_picker/time_picker.dart` — form.save() added
- `lib/core/services/timezone_service.dart` — hotfix: double timezone conversion (commit 34ef471c, folded into eaa2b94e)

## Adaptations

- Plan specified `PopScope(canPop: false)` as primary fix — NOT applied. Investigation revealed the actual root cause was the callback order (`onModeChange(null)` before `onTimeChanged`). `barrierDismissible: false` was already present in `showTimePicker`.
- Plan specified a `mounted` guard — NOT added. The callback order fix addresses the same timing scenario.
- Unplanned hotfix: double timezone conversion in `timezone_service.dart` discovered during manual validation — fixed in commit 34ef471c (folded into main fix commit).
