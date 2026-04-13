# Task: Update destination pages to read cubit from registry

**Plan**: Fix GoRouter Cubit Extras Serialization Warning
**Task ID**: task-03
**Task Path**: task-03-destination-pages
**Depends On**: task-01-cubit-registry
**JIRA**: N/A

## Objective

Update the 6 destination pages that currently receive `TicketDetailsCubit` via constructor to instead look it up from `TicketCubitRegistry` using the `ticketId` they now receive. Constructors change from accepting a cubit to accepting an `int ticketId`.

## Context

After task-01, `TicketCubitRegistry.instance.get(ticketId)` returns the live `TicketDetailsCubit` for any open ticket detail page. Child pages can safely call this in their `build()` or `initState()` because they're always on top of (i.e. navigated from) the ticket detail page that owns the cubit — the cubit is guaranteed to be alive while child pages are visible.

**Pages to update**:

| Page | File | How cubit was used |
|------|------|--------------------|
| `CreateNotePage` | `ticket_details/create_note/presentation/create_note_page.dart` | Passed to `CreateNoteCubit` and as `ticketDetailsCubit:` param |
| `EditCustomFieldsPage` | `ticket_custom_fields/edit_custom_fields/presentation/edit_custom_fields_page.dart` | Cast state as `TicketDetailsLoaded` to build cubit + view |
| `TimerEntriesPage` | `timer_entries/presentation/timer_entries_page.dart` | Passed to `TimerEntriesCubit` and to `TimerEntriesView` |
| `TicketChargesPage` | `ticket_charges/presentation/ticket_charges_page.dart` | Passed to `TicketChargesView` |
| `AddEditTimerEntryPage` | `timer_entries/add_edit_timer_entry/presentation/add_edit_timer_entry_page.dart` | Passed to `AddEditTimerEntryCubit` |
| `AddTicketChargePage` | `ticket_charges/add_ticket_charge/presentation/add_ticket_charge_page.dart` | Passed to `AddTicketChargeView` |

**Pattern to apply for each page**:
1. Change constructor parameter from `required this.ticketDetailsCubit` to `required this.ticketId`
2. In `build()` (or `createState()`/`initState()` for StatefulWidgets), call:
   ```dart
   final ticketDetailsCubit = TicketCubitRegistry.instance.get(ticketId)!;
   ```
3. Continue passing the retrieved cubit to child cubits/views as before — no changes needed deeper in the stack

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-01-cubit-registry is complete (check its `status.md`)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read each of the 6 destination page files in full before changing them
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/ticket/ticket_details/create_note/presentation/create_note_page.dart` | modify | Replace cubit param with ticketId; get cubit from registry |
| `lib/features/ticket/ticket_custom_fields/edit_custom_fields/presentation/edit_custom_fields_page.dart` | modify | Replace cubit param with ticketId; get cubit from registry |
| `lib/features/ticket/timer_entries/presentation/timer_entries_page.dart` | modify | Replace cubit param with ticketId; get cubit from registry |
| `lib/features/ticket/ticket_charges/presentation/ticket_charges_page.dart` | modify | Replace cubit param with ticketId; get cubit from registry |
| `lib/features/ticket/timer_entries/add_edit_timer_entry/presentation/add_edit_timer_entry_page.dart` | modify | Replace cubit param with ticketId; get cubit from registry |
| `lib/features/ticket/ticket_charges/add_ticket_charge/presentation/add_ticket_charge_page.dart` | modify | Replace cubit param with ticketId; get cubit from registry |

### Do NOT Modify

- `lib/core/routing/app_router.dart` — owned by task-02-router-and-params
- `lib/features/ticket/ticket_details/presentation/widget/tickets_notes_tab_page.dart` — owned by task-04
- `lib/features/ticket/ticket_details/presentation/widget/ticket_detail_tab_page.dart` — owned by task-04
- Any `*_view.dart` that CALLS into these pages — owned by task-04

## Implementation Steps

### Step 1: `CreateNotePage`

Read the file. Change constructor:
```dart
// Before
CreateNotePage({required this.ticketId, required this.ticketDetailsCubit});
final TicketDetailsCubit ticketDetailsCubit;

// After
CreateNotePage({required this.ticketId});
```

In `build()`, retrieve cubit before the `BlocProvider`:
```dart
final ticketDetailsCubit = TicketCubitRegistry.instance.get(ticketId)!;
```
Pass the retrieved `ticketDetailsCubit` to `CreateNoteCubit(...)` as before.

### Step 2: `EditCustomFieldsPage`

Read the file. Change constructor to accept only `int ticketId`. In `build()`:
```dart
final ticketDetailsCubit = TicketCubitRegistry.instance.get(ticketId)!;
final state = ticketDetailsCubit.state as TicketDetailsLoaded;
```
Pass `state` to `EditCustomFieldCubit` and to the view as it was before.

### Step 3: `TimerEntriesPage`

Read the file. Change constructor to accept only `int ticketId`. In `build()` (or `createState()`), retrieve:
```dart
final ticketDetailsCubit = TicketCubitRegistry.instance.get(ticketId)!;
```
Pass to `TimerEntriesCubit(ticketDetailsCubit: ticketDetailsCubit)` and to `TimerEntriesView(ticketDetailsCubit: ticketDetailsCubit)` as before.

### Step 4: `TicketChargesPage`

Read the file. Change constructor to accept only `int ticketId`. In `build()`:
```dart
final ticketDetailsCubit = TicketCubitRegistry.instance.get(ticketId)!;
```
Pass to `TicketChargesView(ticketDetailsCubit: ticketDetailsCubit, ...)` as before.

### Step 5: `AddEditTimerEntryPage`

Read the file. The constructor currently takes `params: AddEditTimerEntryPageParams` and `ticketDetailsCubit`. After task-02, `AddEditTimerEntryPageParams` now has `ticketId` instead of `ticketDetailsCubit`.

Update the page:
1. Remove the `ticketDetailsCubit` constructor parameter
2. In `build()`, retrieve it:
   ```dart
   final ticketDetailsCubit = TicketCubitRegistry.instance.get(params.ticketId)!;
   ```
3. Pass to `AddEditTimerEntryCubit(ticketDetailsCubit: ticketDetailsCubit, ...)` as before.

### Step 6: `AddTicketChargePage`

Read the file. After task-02, `AddTicketChargePageParams` has `ticketId` and `ticketChargesCubit` (no `ticketDetailsCubit`).

Update the page:
1. Remove any direct reference to `params.ticketDetailsCubit`
2. In `build()`, retrieve it:
   ```dart
   final ticketDetailsCubit = TicketCubitRegistry.instance.get(params.ticketId)!;
   ```
3. Pass to `AddTicketChargeView(ticketDetailsCubit: ticketDetailsCubit, ticketChargesCubit: params.ticketChargesCubit)` as before.

## Testing

- [ ] `flutter analyze` passes for all 6 modified files
- [ ] Run the app and manually navigate to each sub-route to confirm they load with the correct ticket data
- [ ] If any of these pages have widget tests, update them to provide a mock `TicketCubitRegistry` or use a test helper
- [ ] All existing unit tests for affected cubits (`CreateNoteCubit`, `TimerEntriesCubit`, `AddEditTimerEntryCubit`) still pass
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required for this task

## Completion Criteria

- [ ] All 6 destination pages no longer accept `TicketDetailsCubit` in their constructors
- [ ] All 6 pages retrieve the cubit from `TicketCubitRegistry.instance.get(ticketId)!`
- [ ] Existing cubit tests for `CreateNoteCubit`, `TimerEntriesCubit`, `AddEditTimerEntryCubit` still pass
- [ ] `flutter analyze` clean
- [ ] Status updated in `status.md`
- [ ] Changes committed to `plan/fix-gorouter-cubit-extras/task-03-destination-pages` branch
