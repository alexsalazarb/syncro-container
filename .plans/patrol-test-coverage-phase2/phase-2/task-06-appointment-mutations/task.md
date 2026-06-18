# Task 06 — Appointment mutations

| Field | Value |
|-------|-------|
| **Task ID** | task-06 |
| **Phase** | 2 — Mutations |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-06-appointment-mutations` |
| **JIRA** | N/A |

---

## Objective

Create `appointment_mutations_test.dart` to test appointment create + save. The existing `appointments_test.dart` opens the create form but dismisses without saving.

---

## File Ownership

**Create:**
- `syncro-flutter/integration_test/patrol/features/appointment_mutations_test.dart`

**Do NOT modify:**
- `appointments_test.dart`

---

## Steps

1. Read `appointment_create_page.dart` / `appointment_create_view.dart` to identify required fields and the save button
2. Navigate: Home → Appts tab → create button (IconButton in AppBar)
3. On `appointment_create_page`:
   - Fill title/subject field with `[TEST] Integration test appointment`
   - Fill date/time if required (use current date + 1 day)
   - Fill any other required fields (technician, customer may be pre-populated)
4. Tap Save
5. Assert: redirected to appointments list OR success snackbar visible
6. Navigate back to verify the list is still visible

### Notes on required fields
- Read the form view to find `TextFormField` validators — these indicate required fields
- If technician or customer are required selectors, tap them and select the first available option
- Use soft guard on the create button in case the AppBar doesn't have one

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/appointment_mutations_test.dart --flavor qa -d {device}`

Expected: appointment is created, success state visible.
