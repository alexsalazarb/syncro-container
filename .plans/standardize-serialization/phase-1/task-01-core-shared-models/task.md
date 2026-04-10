# Task: Core Shared Domain Models — fromMap → fromJson

**Plan**: Standardize Domain Model Serialization (fromJson / toJson)
**Phase**: 1
**Task ID**: task-01
**Task Path**: phase-1/task-01-core-shared-models
**Depends On**: None
**JIRA**: N/A

## Objective

Rename all `fromMap` / `toMap` / `fromJson(String)` occurrences in the shared Customer, Contact, Address, CustomerOverview, TinyContact, Settings, TicketSettings, and LocaleData models — and update every call site in infrastructure deserializers, tests, and cross-domain domain files (e.g. `asset.dart`).

## Context

These are the most widely-used domain models in the app. `Customer`, `Contact`, and `Address` are embedded inside `Asset`, `Ticket`, `Appointment`, and many deserializers. Migrating these first (Phase 1) avoids conflicts in the parallel Phase 2 tasks, which all depend on the shared Customer/Contact types already using the canonical name.

The `fromJson(String source)` pattern on Customer, Contact, Address, CustomerOverview, and TinyContact must be **removed entirely** — find every caller of `X.fromJson(rawString)` and replace it with `X.fromJson(jsonDecode(rawString) as Map<String, dynamic>)`.

Convention reference: `syncro-flutter/.cursor/rules/flutter-architecture.mdc` — Domain Model Serialization section.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read `syncro-flutter/lib/features/customers/domain/customer.dart` — understand all 4 classes (Customer, Address, CustomerOverview, Contact) and which have `fromMap`, `fromJson(String)`, and `toMap`
- [ ] Read `syncro-flutter/lib/features/customers/domain/tiny_contact.dart`
- [ ] Read `syncro-flutter/lib/features/settings/locale/domain/settings.dart` and `locale.dart`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/customers/domain/customer.dart` | modify | Rename fromMap → fromJson, remove fromJson(String) on Customer, Address, CustomerOverview, Contact; rename toMap → toJson where present |
| `syncro-flutter/lib/features/customers/domain/tiny_contact.dart` | modify | Rename TinyContact.fromMap → fromJson; remove TinyContact.fromJson(String) |
| `syncro-flutter/lib/features/settings/locale/domain/locale.dart` | modify | Rename LocaleData.fromMap → fromJson |
| `syncro-flutter/lib/features/settings/locale/domain/settings.dart` | modify | Rename Settings.fromMap, TicketSettings.fromMap, TicketType.fromMap → fromJson; rename toMap → toJson where present |
| `syncro-flutter/lib/features/assets/domain/asset.dart` | modify | Call site: `Customer.fromMap(x)` → `Customer.fromJson(x)`; `Contact.fromMap(x)` → `Contact.fromJson(x)` |
| `syncro-flutter/lib/features/customers/infrastructure/customers_deserializer.dart` | modify | Update all `.fromMap(` call sites |
| `syncro-flutter/lib/features/customers/infrastructure/customers_list_deserializer.dart` | modify | Update all `.fromMap(` call sites |
| `syncro-flutter/lib/features/customers/infrastructure/customer_by_id_deserializer.dart` | modify | Update all `.fromMap(` call sites |
| `syncro-flutter/lib/features/end_user/core/infrastructure/contacts_list_deserializer.dart` | modify | Update all `.fromMap(` call sites |
| `syncro-flutter/lib/features/end_user/core/infrastructure/end_users_repository_impl.dart` | modify | Update all `.fromMap(` call sites |
| `syncro-flutter/lib/features/settings/locale/infrastructure/get_settings_deserializer.dart` | modify | Update all `.fromMap(` call sites |
| `syncro-flutter/test/features/customers/domain/customer_test.dart` | modify | Rename test call sites: fromMap → fromJson, fromJson(String) → fromJson(jsonDecode(...)), toMap → toJson |
| `syncro-flutter/test/features/customers/domain/contact_test.dart` | modify | Same rename pattern for Contact and TinyContact |
| `syncro-flutter/test/features/settings/locale/domain/settings_test.dart` | modify | Rename fromMap → fromJson, toMap → toJson |
| `syncro-flutter/test/features/settings/locale/domain/locale_test.dart` | modify | Rename fromMap → fromJson |

### Do NOT Modify

- Any file under `features/ticket/` — owned by task-02 and task-03
- Any file under `features/appointments/` — owned by task-04
- Any remaining infrastructure files not listed above — owned by task-05

## Implementation Steps

### Step 1: Rename in `customer.dart`

For each of the 4 classes in this file (`Customer`, `Address`, `CustomerOverview`, `Contact`):

1. Find `factory X.fromMap(Map<String, dynamic> map)` → rename parameter to `json`, rename factory to `fromJson`
2. Find `factory X.fromJson(String source)` → **delete this factory entirely**, then search for every call site (use grep) and replace `X.fromJson(rawString)` with `X.fromJson(jsonDecode(rawString) as Map<String, dynamic>)`. Add `import 'dart:convert';` where needed.
3. Find `Map<String, dynamic> toMap()` → rename to `toJson()`; update any `copyWith` or internal usages

### Step 2: Rename in `tiny_contact.dart`

1. `TinyContact.fromMap(...)` → `TinyContact.fromJson(...)`
2. `TinyContact.fromJson(String source)` → delete; update call sites using `jsonDecode()`
3. `TinyContact toMap()` → rename to `toJson()` if present

### Step 3: Rename in `locale.dart`

1. `LocaleData.fromMap(...)` → `LocaleData.fromJson(...)`
2. No `fromJson(String)` variant expected, but verify with grep before proceeding

### Step 4: Rename in `settings.dart`

1. `Settings.fromMap` → `Settings.fromJson`
2. `TicketSettings.fromMap` → `TicketSettings.fromJson`
3. `TicketType.fromMap` → `TicketType.fromJson` (note: this is the `TicketType` inside settings.dart, not the one in `ticket_details/domain/ticket_type.dart`)
4. Rename `toMap` → `toJson` on any of the above that have it

### Step 5: Update call sites in `asset.dart`

```dart
// Before
Customer.fromMap(customerMap)
Contact.fromMap(contactMap)

// After
Customer.fromJson(customerMap)
Contact.fromJson(contactMap)
```

### Step 6: Update infrastructure call sites

For each infrastructure file in the File Ownership table:
- `grep -n "fromMap\|toMap\|fromJson(String"` within the file to locate call sites
- Replace `.fromMap(x)` → `.fromJson(x)` for Customer, Contact, Address, CustomerOverview, TinyContact
- Replace `X.fromJson(rawStr)` → `X.fromJson(jsonDecode(rawStr) as Map<String, dynamic>)` if any String-overload calls are found
- Replace `.toMap()` → `.toJson()` if any are found

### Step 7: Update test files

Apply the same rename pattern in each test file:
- `Customer.fromMap(...)` → `Customer.fromJson(...)`
- `.toMap()` → `.toJson()`
- `Customer.fromJson(rawStr)` → `Customer.fromJson(jsonDecode(rawStr) as Map<String, dynamic>)` where applicable

### Step 8: Verify

```bash
cd syncro-flutter
flutter analyze
flutter test test/features/customers/ test/features/settings/
```

Fix any errors before marking complete.

## Testing

- [ ] All tests in `test/features/customers/` pass without modification to test logic (only method renames)
- [ ] All tests in `test/features/settings/` pass
- [ ] `flutter analyze` reports zero new errors
- [ ] No `fromMap` or `toMap` strings remain in any file owned by this task (verify with grep)
- [ ] No `fromJson(String` overload remains in any file owned by this task

## Documentation / KB Updates

- [ ] No new KB doc needed — the convention is already captured in `flutter-architecture.mdc`
- [ ] Run `check-kb-index` if any KB files are changed

## Completion Criteria

- [ ] All `fromMap` → `fromJson` renames complete in owned domain and infrastructure files
- [ ] All `toMap` → `toJson` renames complete in owned files
- [ ] All `fromJson(String source)` factories removed; call sites updated to use `jsonDecode()` inline
- [ ] Tests pass: `flutter test test/features/customers/ test/features/settings/`
- [ ] `flutter analyze` passes with no new issues
- [ ] No regressions in Asset serialization (asset.dart call sites updated)
- [ ] Changes committed to `plan/standardize-serialization/phase-1/task-01-core-shared-models` branch
- [ ] Status updated in `status.md`
