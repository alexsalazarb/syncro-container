# task-12 — Remaining routes

| Field | Value |
|-------|-------|
| **Phase** | 4 |
| **Branch** | `plan/patrol-deep-tests/phase-4/task-12-remaining-routes` |
| **Routes covered** | `barcodeScanner`, `attachmentPreview`, `search`, `alertsDetail`, `addTicketCharge`, `ticketCreateFromChat` |
| **Depends on** | task-01 (some routes accessible from ticket detail) |

---

## Objective

Cover the remaining utility and secondary routes. Create a new test file that exercises each one via its natural entry point.

---

## File Ownership

**Create:**
- `integration_test/patrol/features/utility_screens_test.dart`

---

## Implementation Steps

1. Create `integration_test/patrol/features/utility_screens_test.dart`
2. In the authenticated path, cover each route:

   **Barcode Scanner** (`barcodeScanner`):
   - Find the scan/barcode icon in Assets or Tickets (likely in AppBar)
   - Tap → verify camera permission dialog handled or scanner UI opens
   - Dismiss immediately

   **Attachment Preview** (`attachmentPreview`):
   - Navigate to a ticket with attachments → tap an attachment
   - Verify preview screen opens (image or document viewer)
   - Navigate back

   **Search** (`search`):
   - Find search icon in AppBar on Home or Tickets tab
   - Tap → verify search input appears (`find.byType(TextField)`)
   - Type a query, verify results list renders
   - Clear and dismiss

   **Alerts Detail** (`alertsDetail`):
   - Navigate to Alerts tab → tap an alert item (if any)
   - Verify alert detail screen opens
   - Navigate back

   **Add Ticket Charge** (`addTicketCharge`):
   - Navigate to a ticket detail → Charges tab → add charge button
   - Verify charge form opens (product/labor fields)
   - Dismiss without saving

   **Ticket Create From Chat** (`ticketCreateFromChat`):
   - Navigate to a chat conversation → find "Create Ticket" option (if accessible)
   - Verify ticket create form pre-filled with chat context
   - Dismiss without saving

## Notes

- Guard every step with existence checks — these routes depend on having test data
- Barcode scanner on simulator will not have camera access — assert the scanner widget renders but don't expect camera feed
- If a route can't be reached via natural navigation with current test data, skip it with a comment

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/utility_screens_test.dart \
  --flavor qa -d {device}
```
