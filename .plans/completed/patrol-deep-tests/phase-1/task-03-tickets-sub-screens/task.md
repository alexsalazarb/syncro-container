# task-03 — Tickets: notes, charges, fields, attachments

| Field | Value |
|-------|-------|
| **Phase** | 1 |
| **Branch** | `plan/patrol-deep-tests/phase-1/task-03-tickets-sub-screens` |
| **Routes covered** | `ticketNoteCreate`, `ticketCharges`, `ticketFields`, `ticketAttachments` |
| **Depends on** | task-01 (must be able to reach ticket detail first) |

---

## Objective

From inside a ticket detail screen, verify that the Notes, Charges, Fields, and Attachments sub-tabs render correctly and their associated create/add flows open without submitting.

---

## File Ownership

**Create:**
- `integration_test/patrol/features/ticket_sub_screens_test.dart`

**Do NOT modify:**
- `integration_test/patrol/features/tickets_test.dart`
- `integration_test/patrol/features/ticket_create_test.dart`

---

## Implementation Steps

1. Create `integration_test/patrol/features/ticket_sub_screens_test.dart`
2. Navigate to Tickets tab → tap first list item → reach ticket detail
3. For each sub-tab (Notes, Charges, Fields, Attachments):
   a. Tap the tab label
   b. Wait 1s
   c. Assert the tab content renders (list or empty state)
   d. If there's an add/create button: tap it, verify the create form opens, dismiss without saving
4. Navigate back to list after all assertions

## Key Widgets to Assert

| Sub-screen | Find |
|------------|------|
| Notes tab | `find.text('Notes')` tab + note list or empty state |
| Notes create | `find.text('Add Note')` button or FAB; form with text field |
| Charges tab | `find.text('Charges')` tab + charges list or empty state |
| Fields tab | `find.text('Fields')` tab + custom fields list |
| Attachments tab | `find.text('Attachments')` tab + attachment list or empty state |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/ticket_sub_screens_test.dart \
  --flavor qa -d {device}
```

---

## Notes

- All sub-tabs are inside `TicketDetailsPage` — no separate route navigation needed; they're tab switches within the page
- `ticketNoteCreate` IS a separate route — verify it opens via the "Add Note" button
- Guard every assertion with `if ($(finder).exists)` to handle empty test data gracefully
