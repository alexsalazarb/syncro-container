# task-10 — Customers + end-users

| Field | Value |
|-------|-------|
| **Phase** | 4 |
| **Branch** | `plan/patrol-deep-tests/phase-4/task-10-customers-end-users` |
| **Routes covered** | `customerDetail`, `endUserDetail`, `endUserEdit` |
| **Depends on** | task-01 (customer accessed from ticket detail) |

---

## Objective

Replace the shallow `customers_test.dart` stub. Navigate to a customer detail screen (reachable from a ticket detail) and verify it renders. Also verify the end-user detail and edit form.

---

## File Ownership

**Modify:**
- `integration_test/patrol/features/customers_test.dart`

---

## Implementation Steps

1. In the authenticated path:
   a. Navigate to Tickets tab → tap first ticket → reach ticket detail
   b. Find the customer name on the ticket detail (tappable link or "Customer" field)
   c. Tap it to navigate to `customerDetail`
   d. Wait 2s, assert `CustomerDetailPage` renders: customer name, contact info, or asset list
   e. Look for an "End Users" section or tab within customer detail
   f. If end users exist: tap one → verify `endUserDetail` opens
   g. Look for edit button on end user detail → tap → verify `endUserEdit` form → dismiss without saving
   h. Navigate back to home

## Key Widgets to Assert

| Screen | Find |
|--------|------|
| Customer detail | Customer name text, contact fields |
| End user detail | End user name, email or phone |
| End user edit | Form fields, cancel button |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/customers_test.dart \
  --flavor qa -d {device}
```

---

## Notes

- Customer routes are NOT accessible from a nav tab — must reach via ticket detail
- Guard all steps with existence checks — if the test ticket has no customer linked, skip gracefully
