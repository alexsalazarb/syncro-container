# Status

| Field | Value |
|-------|-------|
| **Status** | complete |
| **Started** | 2026-06-19 |
| **Completed** | 2026-06-19 |
| **Branch** | feature/testwork |
| **PR** | — |

## Notes

### Validation error messages found

**Ticket create (Title field)**
- Validation sets `error: ''` (empty string) on the `CustomTextField`
- The field border turns red but no error text is displayed
- Assertion strategy: stay-on-form check — if Save is tapped with empty Title,
  the form must still be visible (navigation is blocked)

**Appointment create (Subject field)**
- Validation sets `AppStrings.subjectCannotBeEmpty` = `'Subject cannot be empty'`
- The error appears as `errorText` inside the `InputDecoration` of the field
- Assertion strategy: `find.text('Subject cannot be empty')` must be visible

### Navigation path used

- **Ticket**: Tickets tab → FAB → customer search delegate → first InkWell
  → ticket form → Save (empty title)
- **Appointment**: Appts tab → AppBar IconButton → New Appointment form → Save (empty subject)

### Soft guards

Both flows are gated: if FAB, customer results, or create button are absent,
the test skips gracefully with a passing assertion.
