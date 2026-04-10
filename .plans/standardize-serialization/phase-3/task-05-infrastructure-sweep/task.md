# Task: Infrastructure & Final Sweep

**Plan**: Standardize Domain Model Serialization (fromJson / toJson)
**Phase**: 3
**Task ID**: task-05
**Task Path**: phase-3/task-05-infrastructure-sweep
**Depends On**: phase-2/task-02-ticket-home-models, phase-2/task-03-ticket-details-models, phase-2/task-04-appointments-misc-models
**JIRA**: N/A

## Objective

Run a codebase-wide grep sweep to find any remaining `fromMap`, `toMap`, or `fromJson(String source)` occurrences that were missed by tasks 01–04, fix them, and confirm the entire codebase is clean. Also handle two non-feature files (`priority.dart`, `storage.dart`) found during audit that contain `fromMap` calls.

## Context

Phases 1 and 2 handled all known domain and infrastructure files. This sweep task is the safety net. Its primary job is:

1. Run `grep -rn "\.fromMap\|\.toMap()\|fromJson(String" syncro-flutter/lib` and triage every remaining hit
2. For each hit: rename or update call site following the established pattern
3. Run the full test suite and `flutter analyze`

Known files to address in this task (found during plan creation but not owned by earlier tasks):
- `syncro-flutter/lib/core/enums/priority.dart` — contains a `fromMap` call
- `syncro-flutter/lib/core/services/storage.dart` — contains a `fromMap` call
- `syncro-flutter/test/features/ticket/ticket_home/domain/get_tickets_params_test.dart` — uses `toMap()` (test call site)

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Confirm all Phase 2 tasks are **complete** (check each `status.md`)
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/enums/priority.dart` | modify | Update `fromMap` call site or rename definition |
| `syncro-flutter/lib/core/services/storage.dart` | modify | Update `fromMap` call site |
| `syncro-flutter/test/features/ticket/ticket_home/domain/get_tickets_params_test.dart` | modify | `toMap` → `toJson` in test call site |
| Any remaining files found by sweep grep | modify | As discovered — document in Adaptations |

### Do NOT Modify

- Any file already confirmed clean by tasks 01–04 (avoid unnecessary diffs)

## Implementation Steps

### Step 1: Run the sweep grep

```bash
cd syncro-flutter

# Find all remaining fromMap definitions and call sites
grep -rn "\.fromMap\b" lib/ test/

# Find all remaining toMap definitions and call sites
grep -rn "\.toMap()" lib/ test/

# Find all remaining fromJson(String) overloads
grep -rn "fromJson(String" lib/
```

Triage each hit:
- Domain model **definition** → rename to `fromJson`/`toJson`
- Call site of a domain model method → update to new name
- Unrelated `fromMap`/`toMap` (e.g., third-party library or local utility with different semantics) → document as intentional exception in `status.md`

### Step 2: Fix `priority.dart` and `storage.dart`

Read each file, identify what type's `fromMap` they are calling, and update to `fromJson`.

### Step 3: Fix remaining test files

For any test file hit in Step 1:
- Apply the standard rename: `.fromMap(` → `.fromJson(`, `.toMap()` → `.toJson()`

### Step 4: Run full test suite

```bash
cd syncro-flutter
flutter test
flutter analyze
```

All tests must pass. Zero new analyzer warnings or errors.

### Step 5: Final verification

```bash
# These should return ZERO results
grep -rn "\.fromMap\b" syncro-flutter/lib/
grep -rn "\.toMap()" syncro-flutter/lib/
grep -rn "fromJson(String" syncro-flutter/lib/features/
```

If any results remain:
- Intentional exceptions (e.g. Hive adapter, third-party) → document in `status.md` with justification
- Missed renames → fix immediately

## Testing

- [ ] `flutter test` passes (full suite, zero failures)
- [ ] `flutter analyze` reports zero new issues
- [ ] Final sweep grep returns zero results in `lib/` (excluding documented exceptions)

## Documentation / KB Updates

- [ ] If any classes had genuinely different serialization semantics that required an intentional exception to the rule, document them in `docs/kb-container/engineering/best-practices/` with justification
- [ ] Run `check-kb-index` if any KB files are modified

## Completion Criteria

- [ ] `grep -rn "\.fromMap\b" syncro-flutter/lib/` returns zero results (or only documented exceptions)
- [ ] `grep -rn "\.toMap()" syncro-flutter/lib/` returns zero results (or only documented exceptions)
- [ ] `grep -rn "fromJson(String" syncro-flutter/lib/features/` returns zero results
- [ ] Full test suite passes: `flutter test`
- [ ] `flutter analyze` passes with no new issues
- [ ] Changes committed to `plan/standardize-serialization/phase-3/task-05-infrastructure-sweep` branch
- [ ] Status updated in `status.md`
