# Task 09 — Form validation (required fields)

| Field | Value |
|-------|-------|
| **Task ID** | task-09 |
| **Phase** | 3 — Error states & validation |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-09-form-validation` |
| **JIRA** | N/A |

---

## Objective

Create `form_validation_test.dart` to verify that required field validation is shown when forms are submitted empty. Covers ticket create and appointment create forms.

---

## File Ownership

**Create:**
- `syncro-flutter/integration_test/patrol/features/form_validation_test.dart`

**Do NOT modify:**
- `ticket_create_test.dart`
- `appointments_test.dart`
- `appointment_mutations_test.dart`

---

## Steps

### Ticket create — required fields
1. Navigate: Tickets → FAB → customer search → select first customer from search results
   (ticket create requires a customer to open the form)
2. On `ticket_create_page`:
   - Leave the subject/title field empty
   - Tap Save/Submit
3. Assert: validation error visible (red text, helper text, or error snackbar)
4. Dismiss form without saving (`dismissWithLeaveDialog` or back)

### Appointment create — required fields
1. Navigate: Appts tab → create button (AppBar IconButton)
2. On `appointment_create_page`:
   - Leave title/subject empty
   - Tap Save
3. Assert: validation error visible
4. Dismiss without saving

### Notes
- Do NOT submit valid data — validation tests must never mutate backend
- Check the form views for `validator:` callbacks on `TextFormField` to know which fields are required
- If the form requires a customer/technician selector before showing required field errors, fill those selectors first (first available option)

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/form_validation_test.dart --flavor qa -d {device}`

Expected: validation errors visible on both forms; no records created on backend.
