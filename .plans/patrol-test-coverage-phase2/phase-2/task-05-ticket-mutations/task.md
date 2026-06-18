# Task 05 — Ticket mutations

| Field | Value |
|-------|-------|
| **Task ID** | task-05 |
| **Phase** | 2 — Mutations |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-05-ticket-mutations` |
| **JIRA** | N/A |

---

## Objective

Create `ticket_mutations_test.dart` to cover ticket write-path flows:
1. **Create note + save** — open create note form, fill content, submit, verify success
2. **Add charge + save** — open add charge form, fill required fields, submit, verify success

---

## File Ownership

**Create:**
- `syncro-flutter/integration_test/patrol/features/ticket_mutations_test.dart`

**Do NOT modify:**
- `ticket_sub_screens_test.dart`
- `utility_ticket_charges_test.dart`

---

## Steps

### Create note + save
1. Navigate: Tickets → first ticket → Notes tab → FAB
2. Fill the note TextField with `[TEST] Integration test note - can be deleted`
3. Submit (tap Save/Send button)
4. Assert: either redirect back to Notes tab (success) OR success snackbar visible
5. Navigate back to tickets list

### Add charge + save
1. Navigate: Tickets → first ticket → Details tab → Ticket Charges → `ticket_charges_page` → add FAB
2. On `add_ticket_charge_page`:
   - Fill required fields (product/name field with `[TEST] Integration test charge`)
   - Fill quantity (e.g. `1`) and price if required
3. Submit (tap Save)
4. Assert: redirected to `ticket_charges_page` OR success indicator visible
5. Navigate back to tickets list

### Mutation strategy
- Prefix all text with `[TEST]` so QA data can be identified and cleaned manually
- Both tests run against the real QA backend — records WILL persist
- Use soft guards throughout — if FAB/button not found, skip gracefully

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/ticket_mutations_test.dart --flavor qa -d {device}`

Expected: both mutations succeed and app navigates to expected post-save state.
