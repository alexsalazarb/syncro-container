# Task: Update router routes and strip cubit from params classes

**Plan**: Fix GoRouter Cubit Extras Serialization Warning
**Task ID**: task-02
**Task Path**: task-02-router-and-params
**Depends On**: task-01-cubit-registry
**JIRA**: N/A

## Objective

Change 6 affected routes in `app_router.dart` so they no longer pass `TicketDetailsCubit` (or params objects containing it) as `extra`. Remove `ticketDetailsCubit` from the three params classes. After this task the router is clean — destination pages and call sites are updated in tasks 03 and 04.

## Context

The 6 affected routes in `shellBranchTickets` currently use:

| Route | Current extra type | New extra type |
|-------|--------------------|----------------|
| `ticketNoteCreate` | `TicketDetailsCubit` | `int` (ticketId) |
| `ticketFields` | `TicketDetailsCubit` | `int` (ticketId) |
| `timerEntries` | `TicketDetailsCubit` | `int` (ticketId) |
| `ticketCharges` | `TicketChargePageParams` (has cubit) | `int` (ticketId) |
| `timerEntryAddEdit` | `AddEditTimerEntryPageParams` (has cubit) | `AddEditTimerEntryPageParams` (cubit replaced with `int ticketId`) |
| `addTicketCharge` | `AddTicketChargePageParams` (has cubit) | `AddTicketChargePageParams` (cubit replaced with `int ticketId`) |

`TicketChargePageParams` becomes redundant once it only holds `ticketId` — so the route changes to `_createTypedRoute<int>` directly.

**Note**: `AddEditTimerEntryPageParams` still holds `timerEntryParams` (non-cubit data) so the class is kept; only the `ticketDetailsCubit` field is swapped for `int ticketId`.

**Note**: `AddTicketChargePageParams` still holds `ticketChargesCubit` (a screen-scoped cubit, out of scope for this plan). Remove only `ticketDetailsCubit`; add `int ticketId` in its place.

`app_router.dart` is a `part of 'route_cubit.dart'` file — run the app after changes to verify the part file still compiles.

**Relevant files to read first**:
- `lib/core/routing/app_router.dart` lines 332–420 (shellBranchTickets)
- `lib/features/ticket/ticket_charges/presentation/ticket_charges_page.dart` (TicketChargePageParams class)
- `lib/features/ticket/timer_entries/add_edit_timer_entry/presentation/add_edit_timer_entry_page.dart` (AddEditTimerEntryPageParams class)
- `lib/features/ticket/ticket_charges/add_ticket_charge/presentation/add_ticket_charge_page.dart` (AddTicketChargePageParams class)

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-01-cubit-registry is complete (check its `status.md`)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/core/routing/app_router.dart` | modify | Change 6 route type params + childBuilders |
| `lib/features/ticket/ticket_charges/presentation/ticket_charges_page.dart` | modify | Remove `TicketChargePageParams` class (no longer needed) OR simplify it |
| `lib/features/ticket/timer_entries/add_edit_timer_entry/presentation/add_edit_timer_entry_page.dart` | modify | `AddEditTimerEntryPageParams`: replace `TicketDetailsCubit ticketDetailsCubit` with `int ticketId` |
| `lib/features/ticket/ticket_charges/add_ticket_charge/presentation/add_ticket_charge_page.dart` | modify | `AddTicketChargePageParams`: replace `TicketDetailsCubit ticketDetailsCubit` with `int ticketId` |

### Do NOT Modify

- `lib/features/ticket/ticket_details/application/ticket_details_cubit.dart` — owned by task-01
- Any presentation view files (timer_entries_view, ticket_charges_view, etc.) — owned by task-04
- Destination page constructors — owned by task-03

## Implementation Steps

### Step 1: Update `app_router.dart` — 3 direct-cubit routes

Find the three `_createTypedRoute<TicketDetailsCubit>` entries in `shellBranchTickets` and change each:

**ticketNoteCreate** — change type to `int`, update childBuilder:
```dart
_createTypedRoute<int>(
  route: AppRoute.ticketNoteCreate,
  childBuilder: (ticketId) => CreateNotePage(ticketId: ticketId),
),
```

**ticketFields** — change type to `int`, update childBuilder:
```dart
_createTypedRoute<int>(
  route: AppRoute.ticketFields,
  childBuilder: (ticketId) => EditCustomFieldsPage(ticketId: ticketId),
),
```

**timerEntries** — change type to `int`, update childBuilder:
```dart
_createTypedRoute<int>(
  route: AppRoute.timerEntries,
  childBuilder: (ticketId) => TimerEntriesPage(ticketId: ticketId),
),
```

### Step 2: Update `app_router.dart` — ticketCharges route

Remove the `TicketChargePageParams` type. Pass only `int ticketId`:
```dart
_createTypedRoute<int>(
  route: AppRoute.ticketCharges,
  childBuilder: (ticketId) => TicketChargesPage(ticketId: ticketId),
),
```

Update `TicketChargesPage` constructor call to remove `ticketDetailsCubit` — this will cause a compile error until task-03 updates the page constructor. That is expected and resolved in task-03.

### Step 3: Update `app_router.dart` — timerEntryAddEdit route

The params class stays (still carries `timerEntryParams`) but removes cubit:
```dart
_createTypedRoute<AddEditTimerEntryPageParams>(
  route: AppRoute.timerEntryAddEdit,
  childBuilder: (parameters) => AddEditTimerEntryPage(
    params: parameters.timerEntryParams,
    ticketId: parameters.ticketId,
  ),
),
```

### Step 4: Update `app_router.dart` — addTicketCharge route

```dart
_createTypedRoute<AddTicketChargePageParams>(
  route: AppRoute.addTicketCharge,
  childBuilder: (params) => AddTicketChargePage(params: params),
),
```
The route declaration stays the same — only the params class changes.

### Step 5: Update params classes

**`TicketChargePageParams`** (in `ticket_charges_page.dart`):  
Delete the class entirely — it's no longer used after task-04. Or leave it if removing causes cascade errors before task-04 is done. Safe to delete after task-04 merges; for now, leave but note it will be unused.

**`AddEditTimerEntryPageParams`** (in `add_edit_timer_entry_page.dart`):
```dart
class AddEditTimerEntryPageParams {
  AddEditTimerEntryPageParams({
    required this.timerEntryParams,
    required this.ticketId,
  });

  final TimerEntryParams timerEntryParams;
  final int ticketId;
}
```

**`AddTicketChargePageParams`** (in `add_ticket_charge_page.dart`):
```dart
class AddTicketChargePageParams {
  AddTicketChargePageParams({
    required this.ticketId,
    required this.ticketChargesCubit,
  });

  final int ticketId;
  final TicketChargesCubit ticketChargesCubit;
}
```

## Testing

- [ ] `flutter analyze` passes (compile errors in task-03/04 target files are expected and resolved there)
- [ ] Confirm the 6 routes in `shellBranchTickets` no longer reference `TicketDetailsCubit` as a type parameter
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required for this task

## Completion Criteria

- [ ] All 6 routes in `app_router.dart` updated to pass primitive or cubit-free params
- [ ] `AddEditTimerEntryPageParams` and `AddTicketChargePageParams` no longer contain `TicketDetailsCubit`
- [ ] `TicketChargePageParams` removed or marked for removal in task-04
- [ ] `flutter analyze` passes (excluding expected errors in files owned by task-03/04)
- [ ] Status updated in `status.md`
- [ ] Changes committed to `plan/fix-gorouter-cubit-extras/task-02-router-and-params` branch
