# Task: Add priority getters to Ticket model

**Plan**: ticket-priority-domain-getters
**Phase**: 1
**Task ID**: task-01
**Task Path**: task-01-domain-priority-getters
**Depends On**: None
**JIRA**: N/A

## Objective

Add `resolvedPriority` and `priorityLabel` computed getters to the `Ticket` domain model so priority resolution happens once at access time on the model, not repeatedly inside widget `build()` methods.

## Context

Currently `Priority.from(ticket.priority)` and `Priority.labelFrom(ticket.priority)` are called inside `TicketItem.build()` and `DetailTabPageSection.value` — both widget/presentation-layer code. This causes the string-matching logic to run on every frame rebuild, contributing to flicker.

`TicketDetails` extends `Ticket`, so any getter added to `Ticket` is automatically available on `TicketDetails` instances without additional code.

The raw `String? priority` field must remain unchanged because `toJson()` and `copyWith()` rely on the original API string for round-trip serialization (e.g. `"2 Normal"`).

Reference: `syncro-flutter/lib/core/enums/priority.dart` — `Priority.from()` and `Priority.labelFrom()` are pure functions with no side effects.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read `syncro-flutter/lib/core/enums/priority.dart` to confirm `Priority.from` and `Priority.labelFrom` signatures
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart` | modify | Add two computed getters |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/presentation/widget/ticket_item.dart` — owned by task-02-update-presentation-callers
- `syncro-flutter/lib/features/ticket/ticket_details/domain/detail_tab_section.dart` — owned by task-02-update-presentation-callers

## Implementation Steps

### Step 1: Add import for Priority enum

Add the import for the `Priority` enum at the top of `ticket.dart`:

```dart
import 'package:syncro/core/enums/priority.dart';
```

### Step 2: Add computed getters to Ticket class

Add two getters after the field declarations (before `factory Ticket.fromJson`):

```dart
Priority? get resolvedPriority => Priority.from(priority);
String? get priorityLabel => Priority.labelFrom(priority);
```

These are pure derivations from the existing `String? priority` field. They do not affect `Equatable` props, `copyWith`, `fromJson`, or `toJson`.

## Testing

- [ ] `flutter analyze` passes with no new warnings
- [ ] Existing `ticket_test.dart` still passes
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — straightforward getter addition

## Completion Criteria

- [ ] `Ticket` has `resolvedPriority` getter returning `Priority?`
- [ ] `Ticket` has `priorityLabel` getter returning `String?`
- [ ] Getters are pure (no BuildContext, no side effects)
- [ ] Raw `String? priority` field unchanged
- [ ] `fromJson`, `toJson`, `copyWith`, `props` unchanged
- [ ] All existing tests pass
- [ ] Changes committed to `plan/ticket-priority-domain-getters/task-01-domain-priority-getters` branch
- [ ] Status updated in `status.md`
