# Task: Fix `Ticket.fromJson` null cast on `id`/`number`

**Plan**: fix-production-crashes-v180
**Phase**: 2
**Task ID**: task-07
**Task Path**: `phase-2/task-07-ticket-fromjson-nullsafety`
**Depends On**: None
**Ticket**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805)

## Objective

Make `Ticket.fromJson` tolerate a null `id` or `number` from the API instead of throwing a type-cast error that fails the entire tickets-list parse.

## Context

See [investigation.md](../investigation.md) Task 1 for full detail. Root cause: `lib/features/ticket/ticket_home/domain/ticket.dart:53-54` casts `json['id'] as int` and `json['number'] as int` with no null-safety. Real Crashlytics stack trace (Android `f91fb1b40d29c211bdf65655b75937b8`, iOS `50306dfd3ee04e4d8c71ae4d212954dc`) confirms the failure happens on line 54 inside a `compute()` isolate (`get_tickets_deserializer.dart:10`), which fails the whole page of tickets, not just the offending one.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase origin develop`
- [ ] Verify every prerequisite task in `Depends On` is complete (None — no dependencies)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read [investigation.md](../investigation.md) for full root cause context
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/ticket/ticket_home/domain/ticket.dart` | modify | Lines 52-54 — null-safe `id`/`number` |
| `test/features/ticket/ticket_home/domain/ticket_test.dart` | modify | Add regression test cases |

### Do NOT Modify
- `lib/features/ticket/ticket_home/infrastructure/get_tickets_deserializer.dart` — not owned by any task in this plan; referenced for context only, do not modify

## Implementation Steps

### Step 1: Decide the fallback behavior
`id` and `number` are used elsewhere as non-nullable `int` (`Ticket.id`, `Ticket.number` fields, `Equatable` props, `toJson`). Two options:
- **(a)** Keep the fields non-nullable and default to a sentinel (e.g. `0`) when null, logging a non-fatal to Crashlytics via `FirebaseCrashlytics.instance.recordError(..., fatal: false)` — mirrors the `GetTicketsSettingsDeserializer` pattern (`lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart`).
- **(b)** Make the fields nullable (`int?`) and propagate nullability to all call sites that render `ticket.id`/`ticket.number`.

Prefer **(a)** unless a call-site audit shows the ticket is unusable without a real `id`/`number` — a ticket row with `id: 0` should still be filterable/removable rather than crashing the list. Confirm with a quick `rg` for `ticket.id` and `ticket.number` usages before deciding; if any call site does something destructive with `id`/`number` (e.g. uses it as a map key expecting uniqueness), lean toward (a) with a clearly-invalid sentinel and make sure duplicate sentinel rows don't break `Equatable`-based list diffing.

### Step 2: Apply the null-safe cast
Follow the existing style already used on lines 55-59 of the same constructor (`as String?`/`as int?` + `??` fallback), and the `GetTicketsSettingsDeserializer` pattern for how to report a non-fatal to Crashlytics if you choose to record one.

### Step 3: Regression test
Extend `test/features/ticket/ticket_home/domain/ticket_test.dart` with cases that feed `id: null` and `number: null` (separately, and together) and assert the fallback behavior chosen in Step 1 — no exception thrown, and the resulting `Ticket` has the expected sentinel/nullable value.

## Testing

- [ ] New test: `Ticket.fromJson` with `id: null` does not throw
- [ ] New test: `Ticket.fromJson` with `number: null` does not throw
- [ ] Existing tests still pass (`fromJson creates correct Ticket object`, `fromJson handles null user`)
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — this follows an already-documented pattern from `fix-production-crashes-v140`
- [ ] If the fallback approach chosen in Step 1 introduces a new reusable pattern (e.g. a shared "safe int cast + non-fatal report" helper), run `capture-solution`

## Completion Criteria

- [ ] `Ticket.fromJson` no longer throws when `id` or `number` is null
- [ ] Regression tests fail before the fix and pass after
- [ ] Documentation / KB updates completed or explicitly marked not needed
- [ ] Changes committed to `plan/fix-production-crashes-v180/phase-2/task-07-ticket-fromjson-nullsafety` branch
- [ ] Status updated in `status.md`
