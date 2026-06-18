# Task 03 — Ticket secondary flows

| Field | Value |
|-------|-------|
| **Task ID** | task-03 |
| **Phase** | 1 — Missing screens |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-03-ticket-secondary-flows` |
| **JIRA** | N/A |

---

## Objective

Create `ticket_secondary_test.dart` to cover four ticket sub-screens with zero current coverage:
- `timer_entries_page` — per-ticket time entries list (≠ global Time Clock)
- `add_edit_timer_entry_page` — add/edit a timer entry (navigation only, no save)
- `canned_response_create_or_edit_page` — canned responses inside Create Note screen
- `edit_custom_fields_page` — ticket custom fields editor
- `add_ticket_charge_page` — add individual charge (navigation only, no save)

---

## File Ownership

**Create:**
- `syncro-flutter/integration_test/patrol/features/ticket_secondary_test.dart`

**Do NOT modify:**
- `ticket_sub_screens_test.dart`
- `utility_ticket_charges_test.dart`
- Any other test file in this phase

---

## Steps

1. Read `ticket_details_view.dart` to find navigation triggers for timer entries and custom fields
2. Read `create_note_view.dart` to find the canned responses button
3. Read `ticket_charges_page.dart` or `ticket_charges_view.dart` to find the add charge FAB/button
4. Create `ticket_secondary_test.dart` with a single `patrolWidgetTest` covering:

   **Timer entries** (per-ticket):
   - Navigate: Tickets → ticket detail → Details tab → find "Timer Entries" or time section
   - Tap to open `timer_entries_page` → verify it opened
   - Tap add button (FAB) → verify `add_edit_timer_entry_page` opened → navigate back without saving
   - Navigate back to ticket detail

   **Canned responses**:
   - From ticket detail → Notes tab → FAB → `create_note_page`
   - Find canned responses button (icon or "Canned Responses" text)
   - Tap → verify `canned_response_create_or_edit_page` opened → navigate back
   - Navigate back (dismiss note form without saving)

   **Custom fields**:
   - From ticket detail → Details tab → find custom fields section/button
   - Tap → verify `edit_custom_fields_page` opened → navigate back

   **Add ticket charge**:
   - From ticket detail → Details tab → Ticket Charges → `ticket_charges_page`
   - Find add charge FAB or button → tap → verify `add_ticket_charge_page` opened
   - Navigate back without saving

5. All steps behind soft guards — QA data may not expose all sections
6. Run `fvm flutter analyze integration_test/` — must be clean

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/ticket_secondary_test.dart --flavor qa -d {device}`

Expected: test passes; soft-skips sections not present in QA data.
