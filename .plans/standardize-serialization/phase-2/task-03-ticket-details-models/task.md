# Task: Ticket-Details Domain Models — fromMap → fromJson

**Plan**: Standardize Domain Model Serialization (fromJson / toJson)
**Phase**: 2
**Task ID**: task-03
**Task Path**: phase-2/task-03-ticket-details-models
**Depends On**: phase-1/task-01-core-shared-models
**JIRA**: N/A

## Objective

Rename all `fromMap` / `toMap` / `fromJson(String)` occurrences in ticket-details, ticket-charges, worksheets, create-note, canned-responses, and timer-entries domain models; update all corresponding test files and infrastructure deserializers.

## Context

This is the largest group of models to migrate. Key classes with `fromJson(String source)` aliases that must be **removed** (not just renamed):
- `TicketCharge.fromJson(String source)` — wraps `fromMap`
- `TicketLineItem.fromJson(Map<String, dynamic>)` — actually an alias for `fromMap` (same body), keep only one
- `TicketUser.fromJson(String source)` — wraps `fromMap`
- `CreateTimerEntryResponse.fromJson(String source)` — wraps `fromMap`

For each removed `fromJson(String)` factory, find every call site with grep and replace with `X.fromJson(jsonDecode(source) as Map<String, dynamic>)`.

`TicketType` in `ticket_type.dart` has **both** `fromJson(Map)` and `fromMap(Map)` (as an alias). Delete `fromMap`, keep `fromJson`.

This task can run in parallel with task-02 and task-04.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Confirm task-01 is **complete** (check `phase-1/task-01-core-shared-models/status.md`)
- [ ] Read `syncro-flutter/lib/features/ticket/ticket_details/domain/ticket_type.dart` — has both `fromJson` and `fromMap`
- [ ] Read `syncro-flutter/lib/features/ticket/ticket_charges/domain/ticket_charges.dart` — has `fromMap` + `fromJson(String)`
- [ ] Read `syncro-flutter/lib/features/ticket/ticket_details/domain/ticket_user.dart` — has `fromMap` + `fromJson(String)`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/ticket/ticket_details/domain/ticket_type.dart` | modify | Remove `fromMap` alias; keep and retain `fromJson(Map)` |
| `syncro-flutter/lib/features/ticket/ticket_details/domain/ticket_user.dart` | modify | Rename `fromMap` → `fromJson`; remove `fromJson(String)` alias; update callers |
| `syncro-flutter/lib/features/ticket/ticket_details/domain/ticket_line_item.dart` | modify | Remove `fromMap` (keep `fromJson(Map)`); verify the two are identical first |
| `syncro-flutter/lib/features/ticket/ticket_charges/domain/ticket_charges.dart` | modify | Rename `fromMap` → `fromJson`; remove `fromJson(String)` alias; update callers |
| `syncro-flutter/lib/features/ticket/ticket_charges/add_ticket_charge/domain/remove_ticket_charges_response.dart` | modify | `fromMap` → `fromJson` |
| `syncro-flutter/lib/features/ticket/ticket_create/domain/ticket_create_response.dart` | modify | `fromMap` → `fromJson` |
| `syncro-flutter/lib/features/ticket/ticket_details/create_note/domain/create_note_response.dart` | modify | `fromMap` → `fromJson`; remove `fromJson(String)` alias; update callers |
| `syncro-flutter/lib/features/ticket/ticket_details/create_note/canned_responses/domain/canned_response.dart` | modify | `fromMap` → `fromJson` |
| `syncro-flutter/lib/features/ticket/ticket_details/create_note/canned_responses/domain/canned_response_response.dart` | modify | Verify if `fromMap` or `fromJson`; apply canonical rename as needed |
| `syncro-flutter/lib/features/ticket/ticket_details/worksheets/domain/worksheet_template.dart` | modify | `fromMap` → `fromJson`; `toMap` → `toJson` |
| `syncro-flutter/lib/features/ticket/ticket_details/worksheets/domain/worksheet_result.dart` | modify | `WorksheetHistory.fromMap`, `WorksheetContent.fromMap`, `WorksheetResult.fromMap` → `fromJson`; `toMap` → `toJson` |
| `syncro-flutter/lib/features/ticket/timer_entries/add_edit_timer_entry/domain/add_edit_timer_entry_params.dart` | modify | `fromMap` → `fromJson`; `toMap` → `toJson` |
| `syncro-flutter/lib/features/ticket/ticket_create/infrastructure/ticket_create_deserializer.dart` | modify | Update `.fromMap(` call sites |
| `syncro-flutter/lib/features/ticket/ticket_details/worksheets/infrastructure/worksheet_deserializers.dart` | modify | Update `.fromMap(` call sites |
| `syncro-flutter/lib/features/ticket/core/infrastructure/get_search_canned_responses_deserializer.dart` | modify | Update `.fromMap(` call sites |
| `syncro-flutter/lib/features/ticket/ticket_details/create_note/infrastructure/create_timer_entry_deserializer.dart` | modify | Update `.fromMap(` and `fromJson(String)` call sites |
| `syncro-flutter/lib/features/ticket/ticket_charges/add_ticket_charge/infrastructure/remove_ticket_charge_deserializer.dart` | modify | Update `.fromMap(` call sites |
| `syncro-flutter/lib/features/ticket/ticket_charges/search_items/infrastructure/get_items_deserializer.dart` | modify | Update `.fromMap(` call sites |
| `syncro-flutter/test/features/ticket/ticket_details/domain/ticket_user_test.dart` | modify | fromMap → fromJson, fromJson(String) → fromJson(jsonDecode(...)), toMap → toJson |
| `syncro-flutter/test/features/ticket/ticket_details/worksheets/domain/worksheet_result_test.dart` | modify | Same rename pattern |
| `syncro-flutter/test/features/ticket/ticket_details/worksheets/domain/worksheet_template_test.dart` | modify | Same rename pattern |
| `syncro-flutter/test/features/ticket/ticket_details/create_note/canned_responses/domain/canned_response_test.dart` | modify | fromMap → fromJson |
| `syncro-flutter/test/features/ticket/ticket_details/domain/ticket_details_test.dart` | modify | Update any fromMap/toMap call sites in tests |
| `syncro-flutter/test/features/ticket/ticket_charges/add_ticket_charge/domain/remove_ticket_charges_response_test.dart` | modify | fromMap → fromJson |
| `syncro-flutter/test/features/ticket/ticket_details/create_note/domain/create_note_response_test.dart` | modify | fromMap → fromJson, fromJson(String) → fromJson(jsonDecode(...)) |
| `syncro-flutter/test/features/ticket/ticket_create/domain/ticket_create_response_test.dart` | modify | fromMap → fromJson |
| `syncro-flutter/test/features/ticket/ticket_charge/add_ticket_charge/domain/add_ticket_charge_domain_test.dart` | modify | fromMap → fromJson |
| `syncro-flutter/test/features/ticket/timer_entry/domain/add_timer_entry_domain.dart` | modify | fromMap → fromJson, toMap → toJson |
| `syncro-flutter/test/features/ticket/timer_entry_widget/timer_entry_widget_test.dart` | modify | fromMap → fromJson, toMap → toJson |

### Do NOT Modify

- Any file under `features/ticket/ticket_home/` — owned by task-02
- Any file under `features/customers/` — owned by task-01
- Any file under `features/appointments/` — owned by task-04

## Implementation Steps

### Step 1: Resolve dual-factory classes

For each class that has BOTH `fromMap` and `fromJson(Map)` (ticket_type.dart, ticket_line_item.dart):
1. Read both factory bodies — if they are identical, delete `fromMap` and keep `fromJson`
2. If they differ, document the difference in `status.md` (Adaptations) and raise with the team before proceeding

For each class that has `fromMap` + `fromJson(String source)`:
1. Rename `fromMap(Map)` → `fromJson(Map)` (rename parameter to `json`)
2. Delete `fromJson(String source)`
3. Grep for every caller of the `String`-overload and replace with `jsonDecode()` inline

### Step 2: Rename plain `fromMap` → `fromJson`

For remaining classes with only `fromMap` (no `fromJson` conflict):
- Rename parameter name from `map` to `json` for consistency
- Rename factory from `fromMap` to `fromJson`

### Step 3: Rename `toMap` → `toJson`

For `worksheet_template.dart`, `worksheet_result.dart`, `add_edit_timer_entry_params.dart`:
1. Rename `toMap()` → `toJson()`
2. Grep for call sites of `.toMap()` on these specific types and update them

### Step 4: Update infrastructure files

For each infrastructure file listed in File Ownership:
- Replace `.fromMap(x)` → `.fromJson(x)` for affected types
- Replace `X.fromJson(rawString)` → `X.fromJson(jsonDecode(rawString) as Map<String, dynamic>)` where applicable

### Step 5: Update test files

Apply rename pattern in each test file:
- `X.fromMap(...)` → `X.fromJson(...)`
- `.toMap()` → `.toJson()`
- `X.fromJson(rawStr)` → `X.fromJson(jsonDecode(rawStr) as Map<String, dynamic>)` where applicable

### Step 6: Verify

```bash
cd syncro-flutter
flutter analyze
flutter test test/features/ticket/ticket_details/ test/features/ticket/ticket_charges/ test/features/ticket/ticket_create/ test/features/ticket/timer_entry/
```

## Testing

- [ ] All tests in the covered test directories pass
- [ ] `flutter analyze` passes with no new issues
- [ ] `grep -rn "fromMap\|\.toMap()" syncro-flutter/lib/features/ticket/ticket_details/ syncro-flutter/lib/features/ticket/ticket_charges/ syncro-flutter/lib/features/ticket/ticket_create/ syncro-flutter/lib/features/ticket/timer_entries/` returns no results
- [ ] No `fromJson(String` overload remains in any owned domain file

## Documentation / KB Updates

- [ ] No new KB doc needed
- [ ] If a class had genuinely different `fromMap`/`fromJson` semantics, document it in `status.md` Adaptations

## Completion Criteria

- [ ] All `fromMap` factories removed/renamed in owned domain files
- [ ] All `toMap` methods renamed in owned domain files
- [ ] All `fromJson(String)` overloads removed; call sites updated to use `jsonDecode()`
- [ ] Tests pass for all covered areas
- [ ] `flutter analyze` passes with no new issues
- [ ] Changes committed to `plan/standardize-serialization/phase-2/task-03-ticket-details-models` branch
- [ ] Status updated in `status.md`
