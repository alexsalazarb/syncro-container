# Status: Fix `Ticket.fromJson` null cast on `id`/`number`

**Current Status**: complete
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: `plan/fix-production-crashes-v180` (syncro-flutter submodule, based on `develop`) — unified plan branch
**PR**: N/A (PR_INTEGRATION=false) — see plan overview for the current PR link

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-09-07 | not-started | claude | Task created |
| 2026-09-07 | complete | claude | Fix + 3 regression tests, verified red-before/green-after, `flutter analyze` clean, full suite run (1 unrelated pre-existing failure) |

## Summary

Call-site audit (`rg` for `ticket.id`/`ticket.number` across `lib/`) found both fields used as fetch keys and display params (timer entries, appointments, worksheets, ticket details navigation) — no destructive map-key-uniqueness usage. Chose Option (a) from the task's Step 1: sentinel `0` default + non-fatal Crashlytics report, matching `GetTicketsSettingsDeserializer`'s existing pattern, over propagating nullability everywhere.

## Regression Test

Added 3 cases to `test/features/ticket/ticket_home/domain/ticket_test.dart` (`id: null`, `number: null`, both null) — added the same `TestFirebaseCoreHostApi` + Crashlytics method-channel mock setup already used in `get_tickets_settings_deserializer_test.dart` (needed since `Ticket.fromJson` now calls `FirebaseCrashlytics.instance.recordError`). Verified red-before/green-after by stashing the fix and re-running — all 3 new tests failed with the exact original type-cast error before the fix.

## Verification

- [x] `fvm flutter analyze` on changed files: clean, no issues
- [x] `fvm flutter test test/features/ticket/ticket_home/domain/ticket_test.dart`: 16/16 pass
- [x] Full suite (`fvm flutter test`): 2295 tests, 1 failure — same pre-existing `chat_models_test.dart` casing mismatch noted in prior tasks, unrelated, not touched

## Artifacts

- `lib/features/ticket/ticket_home/domain/ticket.dart` — fix
- `test/features/ticket/ticket_home/domain/ticket_test.dart` — 3 new regression tests + Firebase mock setup
- Committed on the unified `plan/fix-production-crashes-v180` branch, pushed to `bla`

## Adaptations

None — task.md's plan and Step 1 guidance matched what the call-site audit found.
