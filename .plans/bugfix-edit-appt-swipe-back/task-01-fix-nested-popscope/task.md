# Task: Remove Duplicate showOnBackDialog from Outer AppScaffold

**Plan**: bugfix-edit-appt-swipe-back
**Task ID**: task-01
**Task Path**: task-01-fix-nested-popscope
**Depends On**: None
**Ticket**: N/A

## Objective

Remove `showOnBackDialog: true` and `backNavigationConfig` from the outer `AppScaffold` in `AppointmentEditPage`, eliminating the double `PopScope(canPop: false)` that blocks back navigation after the Leave Screen dialog is accepted.

## Context

See [investigation.md](../investigation.md) for full root cause details.

`AppointmentEditPage` has two nested `AppScaffold` widgets — both with `showOnBackDialog: true`. Each generates a `PopScope(canPop: false)`. When the swipe-to-back gesture fires, Flutter notifies both PopScope instances. After the inner dialog accepts and calls `routeCubit.goBack()`, the outer PopScope intercepts the resulting pop and blocks navigation.

The outer `AppScaffold` (lines 22–26 in `appointment_edit_page.dart`) exists only to provide a `Scaffold` surface wrapping `MultiBlocProvider`. It must not handle back navigation — only the inner `AppScaffold` (which owns the AppBar) should do that.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify no task is already `in-progress` or `complete` in `status.md`
- [ ] Read [investigation.md](../investigation.md) for full root cause context
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart` | modify | Remove showOnBackDialog + backNavigationConfig from outer AppScaffold only |

### Do NOT Modify

- `syncro-flutter/lib/core/global_widgets/app_scaffold.dart` — reference only
- `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart` — reference only
- Any file not listed above

## Implementation Steps

### Step 1: Open the file

Open `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart`.

### Step 2: Remove the spurious parameters from the outer AppScaffold

The outer `AppScaffold` starts at line 22. Remove `showOnBackDialog: true` and the entire `backNavigationConfig` named argument. Leave all other parameters intact.

**Before** (lines 22–26):
```dart
return AppScaffold(
  showOnBackDialog: true,
  backNavigationConfig: const BackNavigationConfig(
    type: BackNavigationType.normal,
  ),
  body: MultiBlocProvider(
```

**After**:
```dart
return AppScaffold(
  body: MultiBlocProvider(
```

The inner `AppScaffold` (inside the `Builder`, around line 61) keeps its `showOnBackDialog: true` and `backNavigationConfig` unchanged.

### Step 3: Verify imports

After the edit, check if `BackNavigationConfig` and `BackNavigationType` are still referenced elsewhere in the file. If the outer removal makes them unused, the Dart analyzer will flag them — remove those imports only if they become unused.

### Step 4: Run analyzer

```bash
cd syncro-flutter
fvm flutter analyze lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart
```

No errors or warnings expected.

## Testing

- [ ] `fvm flutter analyze` on the affected file passes cleanly
- [ ] Manually test on a device or simulator: swipe-to-back on Edit Appointment → dialog appears → tap "Leave Screen" → navigates back ✅
- [ ] Manually confirm Create Appointment swipe-back still works ✅
- [ ] Existing tests pass: `fvm flutter test test/features/appointments/` (if any)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

No KB/doc updates required — this is a one-line structural fix.

## Completion Criteria

- [ ] Outer `AppScaffold` in `AppointmentEditPage` has no `showOnBackDialog` or `backNavigationConfig` arguments
- [ ] Inner `AppScaffold` is unchanged
- [ ] `fvm flutter analyze` passes
- [ ] Swipe-back on Edit Appointment navigates correctly (manual verification)
- [ ] Changes committed to `plan/bugfix-edit-appt-swipe-back/task-01-fix-nested-popscope` branch
- [ ] Status updated to `complete` in `status.md`
