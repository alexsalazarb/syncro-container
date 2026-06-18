# task-02 — Tickets: create form rendering

| Field | Value |
|-------|-------|
| **Phase** | 1 |
| **Branch** | `plan/patrol-deep-tests/phase-1/task-02-tickets-create-form` |
| **Routes covered** | `ticketCreate` |
| **Depends on** | None (independent of task-01) |

---

## Objective

Verify the ticket creation form opens, renders all expected fields, and can be dismissed — without submitting (no backend mutation).

---

## File Ownership

**Create:**
- `integration_test/patrol/features/ticket_create_test.dart`

**Do NOT modify:**
- `integration_test/patrol/features/tickets_test.dart`
- Any `lib/` source files

---

## Implementation Steps

1. Create `integration_test/patrol/features/ticket_create_test.dart`
2. In the authenticated path:
   a. Navigate to Tickets tab
   b. Find and tap the create/FAB button (likely `find.byType(FloatingActionButton)` or an icon button in the AppBar)
   c. Wait 2s for the create form to load
   d. Assert key form fields exist:
      - Customer field (text input or dropdown)
      - Subject field
      - Problem type / ticket type selector
      - Priority selector
   e. Dismiss the form: tap the back button or close icon — do NOT tap Save/Submit
   f. Assert we're back on the tickets list

## Key Widgets to Assert

| Screen | Widget / Text to find |
|--------|----------------------|
| Create form | `find.text('Create Ticket')` or `find.text('New Ticket')` in AppBar |
| Form fields | `find.text('Subject')`, `find.text('Customer')`, or `find.byType(TextFormField)` |
| After dismiss | `find.text('Tickets')` in nav bar |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/ticket_create_test.dart \
  --flavor qa -d {device}
```

---

## Notes

- The create button location may vary — check `TicketPage`'s AppBar actions or FAB
- If the create route requires selecting a customer first (multi-step), just assert the first step and cancel
- Never tap the submit/save button in integration tests
