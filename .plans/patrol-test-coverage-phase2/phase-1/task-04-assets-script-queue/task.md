# Task 04 — Assets: add script to queue

| Field | Value |
|-------|-------|
| **Task ID** | task-04 |
| **Phase** | 1 — Missing screens |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-04-assets-script-queue` |
| **JIRA** | N/A |

---

## Objective

Extend `assets_test.dart` to cover `add_script_to_queue_page`. The existing test reaches the Scripts tab and opens `script_output_page`, but does not tap the "Add to Queue" action.

---

## File Ownership

**Modify:**
- `syncro-flutter/integration_test/patrol/features/assets_test.dart`

**Do NOT modify:**
- Any other test file in this phase

---

## Steps

1. Read current `assets_test.dart` — it opens Scripts tab and taps a script to see output
2. Read `scripts_page.dart` / `script_output_page.dart` to find the "Add to Queue" button/action
3. Extend the test: after verifying the script output page, look for an "Add to Queue" button
4. If found, tap → verify `add_script_to_queue_page` opened → navigate back WITHOUT submitting
5. Wrap in soft guard (feature-flag dependent — the Scripts tab may not exist)
6. Run `fvm flutter analyze integration_test/` — must be clean

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/assets_test.dart --flavor qa -d {device}`

Expected: existing test still passes; new path soft-skips if add-to-queue button absent.
