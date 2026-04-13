# Task: Unit tests for Ticket priority getters

**Plan**: ticket-priority-domain-getters
**Phase**: 3
**Task ID**: task-03
**Task Path**: task-03-unit-tests
**Depends On**: task-01-domain-priority-getters
**JIRA**: N/A

## Objective

Add unit tests for the new `resolvedPriority` and `priorityLabel` computed getters on the `Ticket` model, covering null, empty, known, and unknown priority values.

## Context

Task-01 adds two getters to `Ticket`:
- `Priority? get resolvedPriority` — delegates to `Priority.from(priority)`
- `String? get priorityLabel` — delegates to `Priority.labelFrom(priority)`

Existing tests in `ticket_test.dart` cover `fromJson`, `props`, and null user handling but do not test any derived getters. The `Priority.from` and `Priority.labelFrom` static methods already have comprehensive tests in `test/core/enums/priority_test.dart` — these new tests verify the getter wiring, not the enum logic itself.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-01 (`task-01-domain-priority-getters`) is complete (check its `status.md`)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read existing `syncro-flutter/test/features/ticket/ticket_home/domain/ticket_test.dart` for test conventions
- [ ] Read existing `syncro-flutter/test/core/enums/priority_test.dart` for Priority test coverage
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/test/features/ticket/ticket_home/domain/ticket_test.dart` | modify | Add test group for priority getters |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart` — owned by task-01
- `syncro-flutter/test/core/enums/priority_test.dart` — existing tests, not this task's scope

## Implementation Steps

### Step 1: Add test group for resolvedPriority

Add a new `group('resolvedPriority')` inside the existing `group('Ticket')` block with these test cases:

1. **Known priority values**: `'urgent'` → `Priority.urgent`, `'high'` → `Priority.high`, `'normal'` → `Priority.normal`, `'low'` → `Priority.low`
2. **Null priority**: `Ticket(priority: null)` → `resolvedPriority` is `null`
3. **Empty priority**: `Ticket(priority: '')` → `resolvedPriority` is `null`
4. **Unknown priority**: `Ticket(priority: 'unknown')` → `resolvedPriority` is `null`
5. **API format priority**: `Ticket(priority: '2 Normal')` → `resolvedPriority` is `Priority.normal`

### Step 2: Add test group for priorityLabel

Add a new `group('priorityLabel')` with these test cases:

1. **Known values**: `'high'` → `'High'`, `'urgent'` → `'Urgent'`, etc.
2. **Null priority**: returns `null`
3. **Unknown priority**: `'unknown'` → returns `'unknown'` (passthrough from `Priority.labelFrom`)
4. **API format**: `'2 Normal'` → `'Normal'`

### Step 3: Add import

Add the Priority import at the top of the test file:

```dart
import 'package:syncro/core/enums/priority.dart';
```

## Testing

- [ ] `flutter test test/features/ticket/ticket_home/domain/ticket_test.dart` — all new tests pass
- [ ] `flutter test` — full suite still passes
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required

## Completion Criteria

- [ ] Test group for `resolvedPriority` covers null, empty, known, unknown, and API-format values
- [ ] Test group for `priorityLabel` covers null, known, unknown, and API-format values
- [ ] All tests pass
- [ ] No regressions in existing functionality
- [ ] Changes committed to `plan/ticket-priority-domain-getters/task-03-unit-tests` branch
- [ ] Status updated in `status.md`
