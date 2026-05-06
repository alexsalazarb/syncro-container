# Investigation: Edit Appointment Swipe-Back Does Not Navigate

**Confidence**: High — root cause confirmed by direct code inspection.
**Investigated**: 2026-05-04

---

## Symptom

On the **Edit Appointment** screen (iOS):
1. Swipe from left edge → "Leave Screen" dialog appears ✅
2. Tap "Leave Screen" button → screen stays on Edit Appointment ❌

On the **Create Appointment** screen the same gesture works correctly.

---

## Origin Analysis

Bug introduced in commit `adfc91d9`:

```
SCLP-690: Mobile App: Show "Leave Screen" confirmation dialog when
navigating back from creation/edit screens
```

When `showOnBackDialog` support was added, the developer correctly configured a single `AppScaffold` in `AppointmentCreatePage`, but incorrectly added `showOnBackDialog: true` to **both** the outer and inner `AppScaffold` in `AppointmentEditPage`.

---

## Root Cause: Double `PopScope(canPop: false)`

### AppScaffold mechanics

`AppScaffold` with `showOnBackDialog: true` wraps its children in:

```dart
PopScope(
  canPop: false,
  onPopInvokedWithResult: (bool didPop, Object? result) async {
    if (didPop) return;
    await _handleOnBack(context); // shows dialog; calls routeCubit.goBack() on accept
  },
  child: child,
)
```

### AppointmentCreatePage (correct — 1 AppScaffold)

```
MultiBlocProvider
  └── AppScaffold(showOnBackDialog: true)  ← single PopScope
        ├── AppBar
        └── AppointmentCreateView
```

Swipe → 1 PopScope fires → dialog → accept → `goBack()` → navigates back ✅

### AppointmentEditPage (buggy — 2 AppScaffolds)

```
AppScaffold(showOnBackDialog: true)          ← outer PopScope  ❌ spurious
  └── MultiBlocProvider
        └── Builder
              └── AppScaffold(showOnBackDialog: true)  ← inner PopScope ✅
                    ├── AppBar
                    └── AppointmentEditView
```

### What happens on swipe

Flutter dispatches `onPopInvokedWithResult(didPop: false)` to **all** registered `PopScope` entries on the route simultaneously (because `canPop: false` for both).

1. Both `_handleOnBack` calls fire — two dialogs appear (or one on top of the other)
2. User accepts the visible (inner) dialog → `routeCubit.goBack()` → `state.pop()` called
3. `state.pop()` triggers another pop attempt
4. The outer `PopScope(canPop: false)` intercepts it → fires `_handleOnBack` again → second "Leave Screen" dialog
5. Navigation never completes — the route stays on screen

The net effect: user sees the dialog, taps "Leave Screen", but cannot leave.

---

## Why Existing Tests Missed This

No widget tests exist for `AppointmentEditPage`'s back navigation flow. The double-AppScaffold structure is a structural mistake invisible to unit tests.

---

## Fix

**Minimal fix** (task-01): Remove `showOnBackDialog: true` and `backNavigationConfig` from the outer `AppScaffold` only. The outer `AppScaffold` exists solely to provide a `Scaffold` surface for `MultiBlocProvider` — it must not intercept back navigation.

```dart
// BEFORE
AppScaffold(
  showOnBackDialog: true,                    // ← remove this
  backNavigationConfig: const BackNavigationConfig(  // ← remove this
    type: BackNavigationType.normal,
  ),
  body: MultiBlocProvider(...)
)

// AFTER
AppScaffold(
  body: MultiBlocProvider(...)
)
```

The inner `AppScaffold` keeps `showOnBackDialog: true` and continues to handle the dialog correctly.

---

## Cross-Project Assessment

Single-project fix. Only `syncro-flutter` is affected. No BE changes required.

---

## Files

| File | Role |
|------|------|
| `syncro-flutter/lib/features/appointments/appointment_edit/presentation/appointment_edit_page.dart` | **Fix target** — lines 22–26 (outer AppScaffold) |
| `syncro-flutter/lib/core/global_widgets/app_scaffold.dart` | Reference — `PopScope` logic |
| `syncro-flutter/lib/features/appointments/appointment_create/presentation/appointment_create_page.dart` | Reference — correct pattern |
