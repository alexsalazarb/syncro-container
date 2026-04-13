# Task: Update widgets to use model getters

**Plan**: ticket-priority-domain-getters
**Phase**: 2
**Task ID**: task-02
**Task Path**: task-02-update-presentation-callers
**Depends On**: task-01-domain-priority-getters
**JIRA**: N/A

## Objective

Replace direct `Priority.from()` / `Priority.labelFrom()` calls in presentation code with the new `Ticket` model getters (`resolvedPriority`, `priorityLabel`), eliminating per-build recomputation.

## Context

After task-01 adds the getters to `Ticket`, two call sites need updating:

1. **`TicketItem.build()`** (lines 36-37) — calls `Priority.from(ticket.priority)` and `Priority.labelFrom(ticket.priority)` on every rebuild
2. **`DetailTabPageSection.priority`** (line 59) — calls `Priority.labelFrom(details.priority)` where `details` is a `TicketDetails` (extends `Ticket`)

**NOT in scope**: `ticket_detail_tab_page.dart` line 357 — that calls `Priority.labelFrom(priorities[index])` on a settings list, not on a `Ticket` field.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-01 (`task-01-domain-priority-getters`) is complete (check its `status.md`)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/ticket/ticket_home/presentation/widget/ticket_item.dart` | modify | Replace Priority.from/labelFrom calls with model getters |
| `syncro-flutter/lib/features/ticket/ticket_details/domain/detail_tab_section.dart` | modify | Replace Priority.labelFrom call with model getter |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/domain/ticket.dart` — owned by task-01-domain-priority-getters
- `syncro-flutter/lib/core/enums/priority.dart` — shared enum, no changes needed

## Implementation Steps

### Step 1: Update TicketItem

In `ticket_item.dart`, replace lines 36-37:

**Before:**
```dart
final matchPriority = Priority.from(ticket.priority);
final priorityLabel = Priority.labelFrom(ticket.priority);
```

**After:**
```dart
final matchPriority = ticket.resolvedPriority;
final priorityLabel = ticket.priorityLabel;
```

Remove the now-unused import `package:syncro/core/enums/priority.dart` if no other reference remains in the file. Check: `Priority.normal` is still used on line 68 as a fallback — so the import stays, but verify.

### Step 2: Update DetailTabPageSection

In `detail_tab_section.dart`, replace line 59:

**Before:**
```dart
final label = Priority.labelFrom(details.priority);
```

**After:**
```dart
final label = details.priorityLabel;
```

Check if the `Priority` import in `detail_tab_section.dart` is still needed for other references. If `Priority` is only used for `labelFrom`, remove the import.

## Testing

- [ ] `flutter analyze` passes with no new warnings
- [ ] Existing `detail_tab_section_test.dart` still passes
- [ ] Visually verify ticket list renders priority chips without flicker (manual)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — straightforward call-site migration

## Completion Criteria

- [ ] `TicketItem.build()` uses `ticket.resolvedPriority` and `ticket.priorityLabel`
- [ ] `DetailTabPageSection.priority` uses `details.priorityLabel`
- [ ] No direct `Priority.from` / `Priority.labelFrom` calls remain for `Ticket`/`TicketDetails` instances
- [ ] Unused imports removed
- [ ] All existing tests pass
- [ ] No regressions in existing functionality
- [ ] Changes committed to `plan/ticket-priority-domain-getters/task-02-update-presentation-callers` branch
- [ ] Status updated in `status.md`
