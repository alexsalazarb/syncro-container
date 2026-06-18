# task-11 — Worksheets

| Field | Value |
|-------|-------|
| **Phase** | 4 |
| **Branch** | `plan/patrol-deep-tests/phase-4/task-11-worksheets` |
| **Routes covered** | `worksheetCreate`, `worksheetDetailEdit`, `worksheetPageHistory` |
| **Depends on** | task-01 (worksheets accessible from ticket detail) |

---

## Objective

Create a new test file for worksheet flows. Worksheets are accessible from ticket detail — verify the worksheet list renders, a worksheet detail opens, and the create flow opens without submitting.

---

## File Ownership

**Create:**
- `integration_test/patrol/features/worksheets_test.dart`

---

## Implementation Steps

1. Create `integration_test/patrol/features/worksheets_test.dart`
2. In the authenticated path:
   a. Navigate to Tickets tab → tap first ticket → reach ticket detail
   b. Find the Worksheets tab or section within ticket detail
   c. Assert worksheet list renders (items or empty state)
   d. If items exist: tap first worksheet → verify `worksheetDetailEdit` opens (form fields for worksheet sections)
   e. Look for page history option (if visible): tap → verify `worksheetPageHistory` renders
   f. Find the create/add worksheet button → tap → verify create form opens → dismiss without saving
   g. Navigate back

## Key Widgets to Assert

| Screen | Find |
|--------|------|
| Worksheet tab | `find.text('Worksheets')` within ticket detail |
| Worksheet detail | Worksheet name, section fields |
| Page history | History entries or version list |
| Create form | `find.text('New Worksheet')` or form fields |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/worksheets_test.dart \
  --flavor qa -d {device}
```
