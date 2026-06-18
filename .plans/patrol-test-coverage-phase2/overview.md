# Plan: Patrol Test Coverage — Phase 2

| Field | Value |
|-------|-------|
| **Slug** | `patrol-test-coverage-phase2` |
| **Type** | Type 2 — Technical (test infrastructure) |
| **Status** | not-started |
| **Project** | syncro-flutter |
| **Branch prefix** | `plan/patrol-test-coverage-phase2` |
| **Dev** | Alex Salazar |
| **QA** | Alex Salazar |
| **Demo date** | No fixed date |
| **Master plan** | None |
| **Created** | 2026-06-18 |

---

## Objective

Expand integration test coverage from ~70% to ~95%+ across three dimensions:

1. **Missing screens** — 11 screens with zero coverage (Phase 1)
2. **Mutations** — all existing tests are read-only; add write-path coverage (Phase 2)
3. **Error states & validation** — no negative-path tests exist yet (Phase 3)

Continues from `patrol-deep-tests` (archived 2026-06-18).

---

## Context

**Existing infrastructure** (`integration_test/patrol/`):
- `utils/app_test_setup.dart` — `startApp()`, `isAuthenticated()`, `withSocketFilter()`, `dismissWithLeaveDialog()`
- 19 tests passing (1 smoke + 18 feature tests) on `feature/testwork`
- `patrolWidgetTest` + `PatrolTester ($)` from `patrol_finders` — do NOT switch to `patrolTest`
- Patrol upgraded to `4.6.1` / `patrol_finders 3.5.0` (2026-06-18)

**Test conventions** (must follow in every task):
- Always wrap in `withSocketFilter(() async { ... })`
- Adaptive auth: `if (isAuthenticated($)) { ... } else { expect($(find.text('Sign In')).exists, isTrue); }`
- Soft guards: `if (!$(widget).exists) { /* soft skip */ return; }` — never fail on missing QA data
- Back navigation: `$(find.byIcon(Icons.arrow_back_ios)).tap()` — app uses `CustomBackIcon`
- List items: `find.descendant(of: find.byType(Scrollable), matching: find.byType(InkWell))`
- Run via: `fvm flutter test integration_test/patrol/features/{test}.dart --flavor qa -d {device}`

**Mutation strategy** (Phase 2):
- Use a `[TEST]` prefix in text fields to identify test-created records
- Assert on success state (e.g. redirect to list, success snackbar, updated list item)
- Do NOT assert on specific IDs or server-assigned values

---

## Phases & Tasks

### Phase 1 — Missing screens (navigation only, no mutations)
| Task | Scope | Files |
|------|-------|-------|
| task-01 | Settings sub-screens: application_lock, locale | `settings_test.dart` (extend) |
| task-02 | End users: detail + edit (via customer detail) | `end_users_test.dart` (new) |
| task-03 | Ticket secondary: timer entries, canned responses, custom fields, add charge | `ticket_secondary_test.dart` (new) |
| task-04 | Assets script queue (add to queue screen) | `assets_test.dart` (extend) |

### Phase 2 — Mutations (write-path)
| Task | Scope | Files |
|------|-------|-------|
| task-05 | Ticket: create note + save, add charge + save | `ticket_mutations_test.dart` (new) |
| task-06 | Appointments: create + save | `appointment_mutations_test.dart` (new) |
| task-07 | Chat: send message; Ticket: add timer entry | `chat_timer_mutations_test.dart` (new) |

### Phase 3 — Error states & validation
| Task | Scope | Files |
|------|-------|-------|
| task-08 | Login: wrong credentials → error visible | `authentication_test.dart` (extend) |
| task-09 | Form validation: ticket create, appointment create required fields | `form_validation_test.dart` (new) |

---

## Kill Criteria

1. **QA backend mutation side effects**: If test-created records break other tests (e.g. pollute list order), pause Phase 2 and add teardown or use a dedicated test account.
2. **Auth failure**: QA token stops working — all authenticated tests fail. Reset credentials before continuing.
3. **Flakiness > 20%**: If >4 of 19 existing tests go intermittently red after changes, stop and diagnose before adding more.

---

## Dependency Graph

```
Phase 1 — all tasks independent of each other (run in parallel)
  task-01 (settings)
  task-02 (end users)
  task-03 (ticket secondary)
  task-04 (assets)

Phase 2 — all tasks independent (run in parallel)
  task-05 (ticket mutations) — no dependency on phase 1
  task-06 (appointment mutations) — no dependency on phase 1
  task-07 (chat + timer entry mutations) — no dependency on phase 1

Phase 3 — after phase 2 (validates that mutations don't break error flows)
  task-08 (login errors)
  task-09 (form validation)
```

---

## Success Criteria

- All 12 missing screens reachable (task-01 through task-04)
- At least 5 write-path flows tested with success assertion (task-05 through task-07)
- Login wrong-credentials error visible (task-08)
- Required field validation visible on at least 2 create forms (task-09)
- 0 regressions in 19 existing passing tests
