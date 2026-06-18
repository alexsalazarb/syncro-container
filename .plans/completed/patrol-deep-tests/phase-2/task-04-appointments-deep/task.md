# task-04 — Appointments: list + detail + create form

| Field | Value |
|-------|-------|
| **Phase** | 2 |
| **Branch** | `plan/patrol-deep-tests/phase-2/task-04-appointments-deep` |
| **Routes covered** | `appts`, `appointmentCreate`, `appointmentUpdate` |
| **Depends on** | None |

---

## Objective

Replace the shallow `appointments_test.dart` stub with deep tests: verify the appointments list renders, navigate into a detail/edit screen, and verify the create form opens with required fields.

---

## File Ownership

**Modify:**
- `integration_test/patrol/features/appointments_test.dart`

---

## Implementation Steps

1. In the authenticated path:
   a. Tap `'Appts'` tab, wait 3s
   b. Assert list renders (list items or empty state)
   c. If items exist: tap first item → verify AppointmentEditPage opens (find `'Save'` button or appointment-specific fields like date/time picker, customer field)
   d. Dismiss edit page without saving
   e. Find the create button in the AppBar (calendar/plus icon) → tap it
   f. Verify `AppointmentCreatePage` opens — assert subject field, date picker, customer field
   g. Dismiss without saving

## Key Widgets to Assert

| Screen | Find |
|--------|------|
| Appts list | `find.text('Appts')` + list or empty state |
| Edit screen | `find.text('Save')` or date/time field |
| Create form | `find.text('New Appointment')` or `find.byType(TextFormField)` |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/appointments_test.dart \
  --flavor qa -d {device}
```
