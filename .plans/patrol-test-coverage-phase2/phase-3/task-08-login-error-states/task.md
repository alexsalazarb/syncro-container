# Task 08 — Login error states

| Field | Value |
|-------|-------|
| **Task ID** | task-08 |
| **Phase** | 3 — Error states & validation |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-08-login-error-states` |
| **JIRA** | N/A |

---

## Objective

Extend `authentication_test.dart` to cover the wrong-credentials error state on the login screen.

Current test only verifies the correct screen is shown based on auth state. It does NOT test error feedback when wrong credentials are entered.

---

## File Ownership

**Modify:**
- `syncro-flutter/integration_test/patrol/features/authentication_test.dart`

**Do NOT modify:**
- Any other test file

---

## Steps

1. Read `login_page.dart` / `login_view.dart` to identify:
   - The email TextField key or label
   - The password TextField key or label
   - The Sign In button
   - The error message widget (Snackbar, Banner, or inline text)

2. Add a second `patrolWidgetTest` case (or extend the existing one with a new flow) for the unauthenticated path:
   - Only run this if `isUnauthenticated($)` — skip silently if already logged in
   - Enter an invalid email: `invalid@test.com`
   - Enter an invalid password: `WrongPassword123!`
   - Tap "Sign In"
   - Wait up to 10 seconds for an error response
   - Assert: an error message is visible (Snackbar with error text, OR inline error widget)

3. Do NOT log in successfully — this test only verifies the error path
4. Run `fvm flutter analyze integration_test/` — clean

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/authentication_test.dart --flavor qa -d {device}`

Expected: error message visible after wrong credentials; app remains on login screen.

---

## Notes

- The app uses OAuth2 (`oauth_webauth`) for production login, but QA flavor may expose a direct credential form — confirm this before implementing
- If OAuth opens a WebView, wrong credentials may not be testable via `patrolWidgetTest` (no access to WebView content) — document this limitation in status.md if encountered
