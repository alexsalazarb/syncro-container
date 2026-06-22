# Status

| Field | Value |
|-------|-------|
| **Status** | adapted |
| **Started** | 2026-06-19 |
| **Completed** | 2026-06-19 |
| **Branch** | `feature/testwork` |
| **PR** | — |

## Adaptation Notes

`ScriptOutputPage` has no "Add to Queue" button or FAB. The `FloatingActionButton` that navigates to `AddScriptToQueuePage` lives on `ScriptsQueuedList` — the default Queued sub-tab within `ScriptsView` (accessed via the Scripts tab on the asset detail page).

**Button found:** `FloatingActionButton` (custom icon via `AppImages.icAdd`) on `ScriptsQueuedList`.

**Verification text:** `'Add to Queue'` — the `AppBar` title of `AddScriptToQueuePage` (sourced from `AppStrings.addToQueue`).

**Test placement:** FAB navigation is exercised immediately after tapping the Scripts tab and before attempting to tap into a finished script item — the guard `$(find.byType(FloatingActionButton)).exists` handles the case where the Queued tab has no FAB (should not happen, but defensive).
