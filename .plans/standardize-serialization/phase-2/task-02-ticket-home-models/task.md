# Task: Ticket-Home Domain Models — fromMap → fromJson

**Plan**: Standardize Domain Model Serialization (fromJson / toJson)
**Phase**: 2
**Task ID**: task-02
**Task Path**: phase-2/task-02-ticket-home-models
**Depends On**: phase-1/task-01-core-shared-models
**JIRA**: N/A

## Objective

Rename all `fromMap` / `toMap` occurrences in the ticket-home domain models (Product, TinyProduct, Ticket, , TicketFilter), update their test files, and update any infrastructure call sites within the ticket-home area.

## Context

Ticket-home models are deserialized from list API responses. `Product` and `TinyProduct` use `fromMap`; `Ticket` and ``use`toMap`(their`fromJson`is already correct).`TicketFilter`has both`fromJson`and`toMap`— only`toMap` needs renaming.

This task can run in parallel with task-03 and task-04 since it only owns files in `ticket/ticket_home/` and `ticket/ticket_home/`-related infrastructure.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Confirm task-01 is **complete** (check `phase-1/task-01-core-shared-models/status.md`)
- [ ] Read `syncro-flutter/lib/features/ticket/ticket_home/domain/product.dart`
- [ ] Read `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart` (verify its `toMap` usage)
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File                                                                                | Action | Notes                                                                        |
| ----------------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------- |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/product.dart`                | modify | `Product.fromMap` → `Product.fromJson`; rename `toMap` → `toJson` if present |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/tiny_product.dart`           | modify | `TinyProduct.fromMap` → `TinyProduct.fromJson`                               |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart`                 | modify | Rename `toMap()` → `toJson()`; `fromJson` already correct                    |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/tickets_settings.dart`       | modify | Rename `toMap()` → `toJson()`; verify `fromMap` if any                       |
| `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket_filter.dart`          | modify | Rename `toMap()` → `toJson()`; `fromJson` already correct                    |
| `syncro-flutter/test/features/ticket/ticket_home/domain/product_test.dart`          | modify | Update `fromMap` → `fromJson`, `toMap` → `toJson` call sites                 |
| `syncro-flutter/test/features/ticket/ticket_home/domain/tickets_settings_test.dart` | modify | Update `toMap` → `toJson` call sites                                         |

### Do NOT Modify

- Any file under `features/ticket/ticket_details/` — owned by task-03
- Any file under `features/ticket/timer_entries/` — owned by task-03
- Any remaining ticket infrastructure not listed above — owned by task-05

## Implementation Steps

### Step 1: Rename in `product.dart`

1. `Product.fromMap(Map<String, dynamic> map)` → rename to `fromJson`, rename parameter to `json`
2. `toMap()` → `toJson()` if present; search for callers with grep and update them

### Step 2: Rename in `tiny_product.dart`

1. `TinyProduct.fromMap(...)` → `TinyProduct.fromJson(...)`
2. Check for `toMap` → rename to `toJson` if found

### Step 3: Rename `toMap` in `ticket.dart`, `tickets_settings.dart`, `ticket_filter.dart`

For each file:

1. `grep -n "toMap"` to locate the method definition
2. Rename the method `toMap()` → `toJson()`
3. Search for callers of `.toMap()` on these types across the codebase; update them

### Step 4: Update test call sites

In each test file, rename:

- `.fromMap(` → `.fromJson(`
- `.toMap()` → `.toJson()`

### Step 5: Verify

```bash
cd syncro-flutter
flutter analyze
flutter test test/features/ticket/ticket_home/
```

## Testing

- [ ] All tests in `test/features/ticket/ticket_home/` pass
- [ ] `flutter analyze` passes with no new issues
- [ ] `grep -rn "fromMap\|\.toMap()" syncro-flutter/lib/features/ticket/ticket_home/` returns no results

## Documentation / KB Updates

- [ ] No KB updates required — convention already documented in `flutter-architecture.mdc`

## Completion Criteria

- [ ] All `fromMap` → `fromJson` renames complete in owned files
- [ ] All `toMap` → `toJson` renames complete in owned files
- [ ] Tests pass: `flutter test test/features/ticket/ticket_home/`
- [ ] `flutter analyze` passes with no new issues
- [ ] Changes committed to `plan/standardize-serialization/phase-2/task-02-ticket-home-models` branch
- [ ] Status updated in `status.md`
