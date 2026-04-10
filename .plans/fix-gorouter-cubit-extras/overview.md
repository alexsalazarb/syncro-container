# Plan: Fix GoRouter Cubit Extras Serialization Warning

**Status**: not-started
**Created**: 2026-04-09
**Last Updated**: 2026-04-09
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A — PR_INTEGRATION=false
**Base Branch**: main

## Objective

Eliminate GoRouter serialization warnings caused by passing `TicketDetailsCubit` (and params classes containing it) as route `extra` values. Cubits cannot be serialized — if the app is killed and restored, these extras are silently dropped. Fix by introducing a `TicketCubitRegistry` so child pages retrieve the live cubit by ticket ID rather than receiving it through the router.

## Scope

### In Scope

- Create `TicketCubitRegistry` — a lightweight in-memory map (`ticketId → TicketDetailsCubit`)
- Wire registry lifecycle into `TicketDetailsCubit` (self-register on init, self-unregister on close)
- Change all 6 affected routes to pass only primitive `int ticketId` (or params without cubit fields)
- Remove `ticketDetailsCubit` field from `TicketChargePageParams`, `AddEditTimerEntryPageParams`, and `AddTicketChargePageParams`
- Update all destination pages to retrieve `TicketDetailsCubit` from the registry
- Update all navigation call sites to pass `ticketId` instead of the cubit

### Out of Scope

- `AddTicketChargePageParams.ticketChargesCubit` — `TicketChargesCubit` is a screen-scoped cubit; fixing it requires either a second registry or a pop-with-result refactor, tracked separately
- Other complex-type GoRouter warnings (e.g. `Customer`, `AssetDetailsParams`, `ChatDetailParameters`) — addressed in a separate plan if needed
- Deep-link restoration for ticket sub-routes — these routes are intentionally not deep-linkable

## Kill Criteria

- If GoRouter v14 deprecates or changes `extra` handling in a way that requires a full routing overhaul, stop and reassess
- If more than 10 additional call sites are discovered that pass `TicketDetailsCubit` not listed here, escalate scope before continuing
- If `TicketDetailsCubit` must be re-fetched (not shared) in sub-routes due to a product change, switch to re-fetch approach instead of registry

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-cubit-registry | Create TicketCubitRegistry + self-register in cubit | not-started | — |
| task-02-router-and-params | Update router routes and strip cubit from params classes | not-started | task-01-cubit-registry |
| task-03-destination-pages | Update destination pages to read cubit from registry | not-started | task-01-cubit-registry |
| task-04-call-sites | Update navigation call sites to pass ticketId | not-started | task-02-router-and-params, task-03-destination-pages |

> Tasks 02 and 03 depend on task 01, but are independent of each other — they can run in parallel.
> Task 04 depends on both 02 and 03.

## Branch Convention

Pattern: `plan/fix-gorouter-cubit-extras/{task-path}`

Examples:
- `plan/fix-gorouter-cubit-extras/task-01-cubit-registry`
- `plan/fix-gorouter-cubit-extras/task-04-call-sites`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `lib/core/routing/app_router.dart` | Router — 6 routes to update |
| `lib/features/ticket/ticket_details/application/ticket_details_cubit.dart` | Add self-registration |
| `lib/features/ticket/shared/ticket_cubit_registry.dart` | **New file** — registry singleton |
| `lib/features/ticket/ticket_details/presentation/widget/ticket_detail_tab_page.dart` | Calls `_handleCustomFields`, `_handleTimerEntries`, `_handleTicketCharges` |
| `lib/features/ticket/ticket_details/presentation/widget/tickets_notes_tab_page.dart` | Calls `_handleCreateNote` |
| `lib/features/ticket/timer_entries/presentation/timer_entries_view.dart` | Pushes `timerEntryAddEdit` |
| `lib/features/ticket/ticket_charges/presentation/ticket_charges_view.dart` | Pushes `addTicketCharge` |
| `lib/features/ticket/ticket_charges/presentation/ticket_charges_page.dart` | `TicketChargePageParams` definition + page |
| `lib/features/ticket/timer_entries/add_edit_timer_entry/presentation/add_edit_timer_entry_page.dart` | `AddEditTimerEntryPageParams` definition + page |
| `lib/features/ticket/ticket_charges/add_ticket_charge/presentation/add_ticket_charge_page.dart` | `AddTicketChargePageParams` definition + page |
| `lib/features/ticket/ticket_details/create_note/presentation/create_note_page.dart` | Destination page |
| `lib/features/ticket/ticket_custom_fields/edit_custom_fields/presentation/edit_custom_fields_page.dart` | Destination page |
| `lib/features/ticket/timer_entries/presentation/timer_entries_page.dart` | Destination page |

## Risks

- `TicketDetailsCubit` closes before child page reads from registry — Mitigated: child pages are always pushed on top of `TicketDetailsPage`, which keeps the cubit alive in the BlocProvider
- Stale registry entry if TicketDetailsPage closes without cubit.close() — Mitigated: cubit self-unregisters in its `close()` override, which is always called by BlocProvider when the widget leaves the tree
- Tests that construct destination pages with cubit in constructor will break — Address in task 03 and 04 by updating test helpers

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] `flutter analyze` runs clean — no new lint warnings
- [ ] GoRouter serialization warning for `TicketDetailsCubit` no longer appears in logs
- [ ] All ticket sub-routes (noteCreate, customFields, timerEntries, charges, addCharge, timerEntryAddEdit) still function correctly
- [ ] Existing unit tests pass
- [ ] `TicketCubitRegistry` is covered by unit tests
