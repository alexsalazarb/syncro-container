# Plan: Patrol Deep Integration Tests

| Field | Value |
|-------|-------|
| **Slug** | `patrol-deep-tests` |
| **Type** | Type 2 — Technical (test infrastructure) |
| **Status** | not-started |
| **Project** | syncro-flutter |
| **Branch prefix** | `plan/patrol-deep-tests` |
| **Dev** | Alex Salazar |
| **QA** | Alex Salazar |
| **Demo date** | No fixed date |
| **Master plan** | None |
| **Created** | 2026-06-10 |

---

## Objective

Expand the Patrol integration test suite from ~14% screen coverage (9 of 50 routes, tab taps only) to deep functional coverage of all remaining routes. Tests must run on a real authenticated QA simulator using `flutter test --flavor qa --concurrency=1`, exercising actual navigation flows, list→detail traversal, and form rendering — without mutating backend data.

---

## Context

**Existing infrastructure** (`integration_test/patrol/`):
- `utils/app_test_setup.dart` — `startApp()`, `isAuthenticated()`, `isUnauthenticated()`, `_appLaunched` guard
- `utils/test_keys.dart` — widget keys for testability
- 13 tests currently passing (1 smoke + 12 feature stubs)

**Test conventions** (must follow in every task):
- `patrol_finders` + `patrolWidgetTest` — NOT `patrol`/`patrolTest` (binding conflict)
- Adaptive auth: always check `isAuthenticated($)` / `isUnauthenticated($)`
- `_appLaunched` guard — never call `app.main()` more than once per binary
- No write operations — open forms, verify fields, cancel/close without submitting
- Run via: `fvm flutter test integration_test/patrol/ --flavor qa -d {device} --concurrency=1`
- Pre-grant permissions: `xcrun simctl privacy {device} grant all com.servably.syncro.mobile.qa`

**Navigation patterns**:
- Tab navigation: `$(find.text('Tickets')).tap()`
- List→detail: `$(find.byType(ListTile)).first.tap()` or `$(find.text(knownItemTitle)).tap()`
- Back navigation: `$(find.byTooltip('Back')).tap()` or `$.pageBack()`
- Wait for screen: `await $.pump(const Duration(seconds: 2))`

---

## Phases & Tasks

### Phase 1 — Ticket flows (highest priority, most routes)
| Task | Scope | Routes |
|------|-------|--------|
| task-01 | Tickets list + detail navigation | `tickets`, `ticketDetail` |
| task-02 | Ticket create form rendering | `ticketCreate` |
| task-03 | Ticket sub-screens | `ticketNoteCreate`, `ticketCharges`, `ticketFields`, `ticketAttachments` |

### Phase 2 — Appointments, Assets, Chat
| Task | Scope | Routes |
|------|-------|--------|
| task-04 | Appointments list + detail + create form | `appts`, `appointmentCreate`, `appointmentUpdate` |
| task-05 | Assets list + detail + script output | `assets`, `assetDetails`, `scriptOutput`, `addToQueue` |
| task-06 | Chat list + conversation | `chats`, `chatDetail`, `cannedResponseCreateOrEdit` |

### Phase 3 — Settings, Time Clock, Notifications
| Task | Scope | Routes |
|------|-------|--------|
| task-07 | Settings all sub-screens | `settings`, `settingsAppLock`, `appearance`, `locale`, `about` |
| task-08 | Time Clock deep | `timeClock`, `timerEntries`, `timerEntryAddEdit` |
| task-09 | Notifications list | `notifications` |

### Phase 4 — Secondary & utility screens
| Task | Scope | Routes |
|------|-------|--------|
| task-10 | Customers + end-users | `customerDetail`, `endUserDetail`, `endUserEdit` |
| task-11 | Worksheets | `worksheetCreate`, `worksheetDetailEdit`, `worksheetPageHistory` |
| task-12 | Remaining routes | `barcodeScanner`, `attachmentPreview`, `search`, `alertsDetail`, `addTicketCharge`, `ticketCreateFromChat` |

---

## Kill Criteria

1. **Auth failure**: QA token expires and auto-refresh stops working — tests permanently fail at launch. Fix token or reset simulator before continuing.
2. **Flakiness > 20%**: If more than 2 of 13 existing tests become intermittently red, pause and diagnose before adding more.
3. **Backend unavailability**: QA backend (`ballastlanedev.syncromsp.com`) becomes unreachable — all authenticated tests fail. Not a code issue; pause work.

---

## Dependency Graph

```
Phase 1 (tickets)
  task-01 ──────────────────┐
  task-02 (independent)     ├── Phase 2 can start in parallel
  task-03 → requires task-01 (navigates from detail)

Phase 2 (appointments, assets, chat) — all independent of each other
  task-04
  task-05
  task-06

Phase 3 (settings, time clock, notifications) — all independent
  task-07
  task-08
  task-09

Phase 4 (secondary) — all independent, some depend on Phase 1 (task-10 may need ticket context)
  task-10
  task-11 → requires task-01 (navigates from ticket detail)
  task-12
```

Phases 1–3 can largely run in parallel across tasks. Phase 4 can start once Phase 1 task-01 is complete.

---

## Success Criteria

- All 50 AppRoute entries covered by at least one test assertion
- 0 regressions in the 13 existing passing tests
- Full suite passes with `All tests passed!` on authenticated QA simulator
- Coverage jumps from ~14% → ~95%+ of routes exercised
