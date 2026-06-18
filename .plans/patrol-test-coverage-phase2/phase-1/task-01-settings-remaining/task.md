# Task 01 — Settings remaining sub-screens

| Field | Value |
|-------|-------|
| **Task ID** | task-01 |
| **Phase** | 1 — Missing screens |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-01-settings-remaining` |
| **JIRA** | N/A |

---

## Objective

Extend `settings_test.dart` to cover the two settings sub-screens currently untested: **Application Lock** and **Language (locale)**.

Current `settings_test.dart` covers: Settings home → About → Appearance → back.
Missing: `application_lock_page`, `locale_page`.

---

## File Ownership

**Modify:**
- `syncro-flutter/integration_test/patrol/features/settings_test.dart`

**Do NOT modify:**
- Any other test file in this phase (all phase-1 tasks are independent)

---

## Steps

1. Read `settings_test.dart` to understand current navigation pattern
2. Locate the "Application Lock" and "Language" row labels in `settings_page.dart` or `settings_view.dart`
3. Add navigation to **Application Lock**:
   - Tap the Application Lock row in settings
   - Verify the screen opened (check for a toggle or lock-related widget)
   - Navigate back
4. Add navigation to **Language**:
   - Tap the Language row in settings
   - Verify the locale page opened (check for language options)
   - Navigate back
5. Both additions must use soft guards (`if ($(find.text('Application Lock')).exists)`)
6. Verify `fvm flutter analyze integration_test/` is clean

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/settings_test.dart --flavor qa -d {device}`

Expected: test passes, both sub-screens are navigated to and back.

---

## Notes

- `unlock_app_page` (PIN/biometric screen on app resume) is intentionally excluded — it requires the app lock to be enabled and the app to be backgrounded, which needs native automation (`patrolTest`). Flag this in a code comment if relevant.
- Application Lock settings page may have a toggle — do NOT tap it (would mutate device state).
