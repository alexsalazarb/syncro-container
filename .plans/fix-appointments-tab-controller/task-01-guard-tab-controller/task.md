# Task: Guard tabController accesses in cubit and view

**Plan**: Fix Appointments TabController LateInitializationError
**Phase**: 1
**Task ID**: task-01
**Task Path**: task-01-guard-tab-controller
**Depends On**: None
**JIRA**: N/A

## Objective

Make `tabController` nullable in `AppointmentsListCubit` and add consistent null/readiness guards to every method that accesses it, so that neither a fresh global cubit (before the view mounts) nor a post-`disposeView()` state can produce a `LateInitializationError`.

## Context

`AppointmentsListCubit` is a global cubit registered at app startup in
`syncro-flutter/lib/app/dependency/app_providers.dart`. Its `TabController` is a UI-lifecycle
object that is only available after `AppointmentsView.initState()` calls `setTabController()`.

Currently, `_tabControllerReady` is used as a guard flag in `tabListener()` and
`_handleFilterChange()`, but the pattern was not applied to `refresh()` or `disposeView()`.
In addition, `AppointmentsView.build()` accesses `listCubit.tabController` three times outside
the guaranteed `initState` path.

The fix keeps `tabController` as `late` but makes it nullable (`TabController?`) so the Dart
type system enforces null-awareness at every call site. The `_tabControllerReady` flag is
retained as a semantic signal for the listener/filter paths that need to distinguish
"not-yet-set" from "set-but-may-have-wrong-state".

Relevant KB: Flutter architecture conventions at
`syncro-flutter/.cursor/rules/flutter-architecture.mdc` — Cubits use `safeEmit()`, states are
Equatable; UI controllers (TabController) should eventually live in `State`, not Cubits (future
tech-debt item, out of scope here).

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read `appointments_list_cubit.dart` and `appointments_view.dart` in full before editing
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/appointments_home/application/appointments_list_cubit.dart` | modify | Make tabController nullable; add guards to refresh() and disposeView() |
| `syncro-flutter/lib/features/appointments/appointments_home/presentation/appointments_view.dart` | modify | Handle nullable tabController in build() and BlocListener |

### Do NOT Modify

- `syncro-flutter/lib/app/dependency/app_providers.dart` — not needed for this fix
- `syncro-flutter/lib/features/appointments/appointments_home/application/appointments_list_state.dart` — no state changes required
- Any paging controller, use case, or filter cubit files

## Implementation Steps

### Step 1: Make `tabController` nullable in the cubit

In `appointments_list_cubit.dart`, change the field declaration:

```dart
// Before
late TabController tabController;

// After
TabController? tabController;
```

`setTabController()` already sets the field; no changes needed there beyond the type:

```dart
void setTabController(TabController controller) {
  tabController = controller..addListener(tabListener);
  _tabControllerReady = true;
}
```

### Step 2: Guard `refresh()` with `_tabControllerReady`

```dart
Future<void> refresh() async {
  if (!_tabControllerReady || tabController == null) return;
  if (tabController!.index == 0) {
    mineController.refresh();
  } else {
    allController.refresh();
  }
}
```

### Step 3: Guard `disposeView()` with `_tabControllerReady`

```dart
void disposeView() {
  if (isClosed) return;
  if (!_tabControllerReady || tabController == null) return;
  tabController!.removeListener(tabListener);
  tabController!.dispose();
  tabController = null;
  _tabControllerReady = false;
}
```

Setting `tabController = null` after disposal makes the guard work correctly on repeated
navigations to/from the Appointments section.

### Step 4: Update `AppointmentsView.build()` for nullable type

`initState()` always runs before the first `build()`, so `tabController` will be non-null when
`build()` executes in the normal flow. However, since the type is now `TabController?`, the
compiler requires null-safe access. Use `!` with a brief comment at each access site:

In `appointments_view.dart` `build()`:

```dart
// tabController is set in initState() before build() runs
AppointmentsTabBar(
  tabController: listCubit.tabController!,
),
...
TabBarView(
  controller: listCubit.tabController!,
  children: pages,
)
```

In the `BlocListener.listener` callback (line ~59):

```dart
listener: (context, state) {
  final targetIndex = state.selectedType == AppointmentType.mine ? 0 : 1;
  final tc = listCubit.tabController;
  if (tc != null && tc.index != targetIndex) {
    tc.animateTo(targetIndex);
  }
},
```

Use a null-safe local variable here (unlike `build()` body) because the listener can fire at any
point after the widget is inserted, including after a `disposeView()` call on navigation away.

### Step 5: Lint and analyze

```bash
cd syncro-flutter && flutter analyze
```

Fix any type errors or warnings introduced by the nullable change.

## Testing

- [ ] Open Appointments section for the first time in a fresh app run — no crash
- [ ] Navigate away from and back to Appointments multiple times — no crash
- [ ] Call `refresh()` before `setTabController()` in a unit test — should return early without error
- [ ] Call `disposeView()` before `setTabController()` in a unit test — should return early without error
- [ ] Existing tests still pass: `cd syncro-flutter && flutter test test/features/appointments/`
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — this is a targeted bug fix with no new architectural patterns

## Completion Criteria

- [ ] `LateInitializationError` no longer thrown when opening Appointments section
- [ ] `refresh()` and `disposeView()` both guard against uninitialized `tabController`
- [ ] `appointments_view.dart` compiles cleanly with nullable `TabController?`
- [ ] All existing appointment tests pass
- [ ] `flutter analyze` passes with no new issues
- [ ] Changes committed to `plan/fix-appointments-tab-controller/task-01-guard-tab-controller` branch
- [ ] Status updated in `status.md`
