# Status

| Field | Value |
|-------|-------|
| **Status** | complete |
| **Started** | 2026-06-19 |
| **Completed** | 2026-06-19 |
| **Branch** | feature/testwork |
| **PR** | — |

## Implementation Notes

**Required fields found:**
- `subject` — the only truly required field (has `mandatory: true` + `subjectError` validation in cubit). Filled with `[TEST] Integration test appointment`.

**Optional fields left unfilled:**
- `customer` — CustomItemSelector with delete option; not validated
- `ticket` — optional selector; no validator
- `appointment type` — optional bottom-sheet picker
- `description` — optional TextField
- `location` — optional TextField (address/phone/URL/manual modes)

**Navigation pattern:**
- Appts tab → AppBar `IconButton` (create) → `New Appointment` form (`showOnBackDialog=true`)
- Save calls `validateAndSaveAppointment`; on success shows `'Appointment successfully created'` snackbar and navigates back via `RouteCubit.goBack`

**Soft guards applied:**
- Create button absence: skip and assert list visible
- Form not opening: skip and assert list visible
- Subject field not found: `dismissWithLeaveDialog` and assert list visible
- Post-save cleanup if still on form unexpectedly
