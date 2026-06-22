# Status

| Field | Value |
|-------|-------|
| **Status** | complete |
| **Started** | 2026-06-19 |
| **Completed** | 2026-06-19 |
| **Branch** | feature/testwork |
| **PR** | — |

## Implementation Notes

### Flow 1 — Chat send message

- **Send button**: `IconButton` with a custom SVG icon (`AppImages.commonSend`), enabled only when `SendButtonCubit` detects non-empty text in the `TextField`. Matched via `find.byWidgetPredicate((w) => w is IconButton && w.onPressed != null)`.
- **Input hint text**: `'Type a message'` (from `AppStrings.chatTypeMessage`).
- **Send assertion**: `controller.clear()` is called on success — check `textField.controller?.text.isEmpty`.
- **Soft guard**: Input field visible only when the authenticated user is the assigned tech (`state.technicianId == user.userId`).

### Flow 2 — Ticket timer entry add + save

- **Form fields discovered**:
  - **Notes** (index 0): `MentionsWidget` renders an internal `TextField`. Filled with `'[TEST] Integration test timer entry'`.
  - **Duration in minutes** (index 1): `CustomTextField` wrapping a `TextField`. Filled with `'15'`.
  - No start/end time fields — new entries default to current time.
- **Save button**: `FilledButton` with text `'Save'` (`AppStrings.save`).
- **Add button**: `IconButton` in AppBar, disabled when a timer is actively running — wrapped in `try/catch`.
- **Success assertion**: navigated back to `Timer Entries` page OR snackbar contains `'successfully'` / `'created'`.
