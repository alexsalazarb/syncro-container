# Task: Update navigation call sites to pass ticketId

**Plan**: Fix GoRouter Cubit Extras Serialization Warning
**Task ID**: task-04
**Task Path**: task-04-call-sites
**Depends On**: task-02-router-and-params, task-03-destination-pages
**JIRA**: N/A

## Objective

Update every `pushRoute` call that currently passes `TicketDetailsCubit` (or a params class containing it) as `parameters:`. Replace with the primitive `ticketId`. This is the final task — after it completes the app compiles cleanly with no GoRouter cubit-extra warnings.

## Context

All call sites are inside `TicketDetailsView`'s sub-widgets, which already have `TicketDetailsCubit` available via `context.read<TicketDetailsCubit>()`. The cubit exposes `ticketId` (or it's available from the loaded state's `ticketDetails.id`).

**Call sites to update**:

| File | Method | Current push | After fix |
|------|--------|-------------|-----------|
| `tickets_notes_tab_page.dart` | `_handleCreateNote` | `parameters: context.read<TicketDetailsCubit>()` | `parameters: context.read<TicketDetailsCubit>().ticketId` |
| `ticket_detail_tab_page.dart` | `_handleCustomFields` | `parameters: context.read<TicketDetailsCubit>()` | `parameters: context.read<TicketDetailsCubit>().ticketId` |
| `ticket_detail_tab_page.dart` | `_handleTimerEntries` | `parameters: context.read<TicketDetailsCubit>()` | `parameters: context.read<TicketDetailsCubit>().ticketId` |
| `ticket_detail_tab_page.dart` | `_handleTicketCharges` | `parameters: TicketChargePageParams(ticketId: ..., cubit: ...)` | `parameters: widget.data.id` (the `int` ticketId) |
| `timer_entries_view.dart` | push `timerEntryAddEdit` | `AddEditTimerEntryPageParams(timerEntryParams: ..., cubit: ...)` | `AddEditTimerEntryPageParams(timerEntryParams: ..., ticketId: widget.ticketDetailsCubit.ticketId)` |
| `ticket_charges_view.dart` | push `addTicketCharge` | `AddTicketChargePageParams(cubit: ..., chargesCubit: ...)` | `AddTicketChargePageParams(ticketId: ..., ticketChargesCubit: ...)` |

**Note on `timer_entries_view.dart` and `ticket_charges_view.dart`**: these views still hold a `TicketDetailsCubit ticketDetailsCubit` constructor field (passed by their parent page). After task-03, the PARENT pages (`TimerEntriesPage`, `TicketChargesPage`) fetch the cubit from the registry and pass it to the view. The views themselves don't change their own constructor — only how they build the params for sub-route pushes.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-02-router-and-params is complete (check its `status.md`)
- [ ] Verify task-03-destination-pages is complete (check its `status.md`)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read each of the 4 files below in full before changing them
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/ticket/ticket_details/presentation/widget/tickets_notes_tab_page.dart` | modify | `_handleCreateNote`: pass `ticketId` not cubit |
| `lib/features/ticket/ticket_details/presentation/widget/ticket_detail_tab_page.dart` | modify | `_handleCustomFields`, `_handleTimerEntries`, `_handleTicketCharges`: pass `ticketId` |
| `lib/features/ticket/timer_entries/presentation/timer_entries_view.dart` | modify | Push to `timerEntryAddEdit` with `AddEditTimerEntryPageParams(ticketId: ...)` |
| `lib/features/ticket/ticket_charges/presentation/ticket_charges_view.dart` | modify | Push to `addTicketCharge` with `AddTicketChargePageParams(ticketId: ...)` |

### Do NOT Modify

- `lib/core/routing/app_router.dart` — owned by task-02-router-and-params (already complete)
- Any destination page — owned by task-03-destination-pages (already complete)
- `lib/features/ticket/ticket_details/application/ticket_details_cubit.dart` — owned by task-01

## Implementation Steps

### Step 1: `tickets_notes_tab_page.dart` — `_handleCreateNote`

```dart
// Before
parameters: context.read<TicketDetailsCubit>(),

// After
parameters: context.read<TicketDetailsCubit>().ticketId,
```

### Step 2: `ticket_detail_tab_page.dart` — three handlers

**`_handleCustomFields`**:
```dart
// Before
parameters: context.read<TicketDetailsCubit>(),

// After
parameters: context.read<TicketDetailsCubit>().ticketId,
```

**`_handleTimerEntries`**:
```dart
// Before
parameters: context.read<TicketDetailsCubit>(),

// After
parameters: context.read<TicketDetailsCubit>().ticketId,
```

**`_handleTicketCharges`**:
```dart
// Before
parameters: TicketChargePageParams(
  ticketId: widget.data.id,
  ticketDetailsCubit: context.read<TicketDetailsCubit>(),
),

// After
parameters: widget.data.id,   // int ticketId directly
```

Remove the `TicketChargePageParams` import if nothing else in the file uses it.

### Step 3: `timer_entries_view.dart` — push to `timerEntryAddEdit`

Read the full push call. Update `AddEditTimerEntryPageParams` to use `ticketId`:
```dart
// Before
parameters: AddEditTimerEntryPageParams(
  timerEntryParams: ...,
  ticketDetailsCubit: widget.ticketDetailsCubit,
),

// After
parameters: AddEditTimerEntryPageParams(
  timerEntryParams: ...,
  ticketId: widget.ticketDetailsCubit.ticketId,
),
```

### Step 4: `ticket_charges_view.dart` — push to `addTicketCharge`

Read the full push call. Update `AddTicketChargePageParams` to use `ticketId`:
```dart
// Before
parameters: AddTicketChargePageParams(
  ticketDetailsCubit: widget.ticketDetailsCubit,
  ticketChargesCubit: context.read<TicketChargesCubit>(),
  ...
),

// After
parameters: AddTicketChargePageParams(
  ticketId: widget.ticketDetailsCubit.ticketId,
  ticketChargesCubit: context.read<TicketChargesCubit>(),
),
```

### Step 5: Final cleanup

- Remove any now-unused imports of `TicketChargePageParams` or direct `TicketDetailsCubit` imports in call-site files (if the cubit is only now read via `context.read`, which is already imported via BlocProvider)
- Run `flutter analyze` to confirm no unused imports remain

## Testing

- [ ] `flutter analyze` passes across the full project
- [ ] Run the app end-to-end:
  - [ ] Create a note from ticket detail → no GoRouter warning in logs
  - [ ] Edit custom fields from ticket detail → no GoRouter warning
  - [ ] Open timer entries → no GoRouter warning
  - [ ] Add/edit a timer entry → no GoRouter warning
  - [ ] Open ticket charges → no GoRouter warning
  - [ ] Add a ticket charge → no GoRouter warning
- [ ] GoRouter warning `An extra with complex data type TicketDetailsCubit` no longer appears in logs
- [ ] All existing tests pass (`flutter test`)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Update `docs/kb-projects/syncro-flutter/README.md` to note the `TicketCubitRegistry` pattern under Technical Architecture, OR run `document-solution` to create a KB doc for the registry pattern
- [ ] Run `check-kb-index` after any KB changes

## Completion Criteria

- [ ] All 4 call-site files updated to pass `ticketId` (int) instead of cubit or params-with-cubit
- [ ] GoRouter warning no longer appears when navigating ticket sub-routes
- [ ] `flutter analyze` clean across the whole project
- [ ] All tests pass
- [ ] KB updated or `document-solution` run
- [ ] Status updated in `status.md`
- [ ] Changes committed to `plan/fix-gorouter-cubit-extras/task-04-call-sites` branch
