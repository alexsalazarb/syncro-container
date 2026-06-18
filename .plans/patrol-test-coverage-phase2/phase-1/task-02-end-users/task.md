# Task 02 — End users (detail + edit)

| Field | Value |
|-------|-------|
| **Task ID** | task-02 |
| **Phase** | 1 — Missing screens |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-02-end-users` |
| **JIRA** | N/A |

---

## Objective

Create `end_users_test.dart` to cover `end_user_detail_page` and `end_user_edit_page`.

These screens are accessible via: Tickets → ticket detail → Details tab → Customer section → CustomerDetailsPage → Contacts/End Users section → end user row.

---

## File Ownership

**Create:**
- `syncro-flutter/integration_test/patrol/features/end_users_test.dart`

**Do NOT modify:**
- `customers_test.dart` (already navigates to CustomerDetailsPage — this task goes deeper)
- Any other test file in this phase

---

## Steps

1. Read `customers_test.dart` for the navigation path to `CustomerDetailsPage`
2. Read `customer_details_view.dart` to find the end user list widget and tap target
3. Create `end_users_test.dart` following the standard template (imports, `setupIntegrationTest`, `patrolWidgetTest`, `withSocketFilter`, `startApp`)
4. Navigation path:
   - Start at Tickets tab
   - Navigate into first ticket → Details tab → Customer section → CustomerDetailsPage
   - Scroll to find the End Users / Contacts section
   - If an end user row exists, tap it → verify `end_user_detail_page` opened
   - Navigate back to CustomerDetailsPage
   - Find the edit/add end user button → tap → verify `end_user_edit_page` opened
   - Dismiss without saving (`dismissWithLeaveDialog` or back arrow)
5. All steps behind soft guards — QA data may not have contacts
6. Run `fvm flutter analyze integration_test/` — must be clean

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/end_users_test.dart --flavor qa -d {device}`

Expected: test passes with soft guards; if no end users exist in QA data, test exits gracefully.

---

## Notes

- `end_user_edit_page` uses `showOnBackDialog=true` (based on similar patterns in the app) — use `dismissWithLeaveDialog($)` from `app_test_setup.dart` to dismiss.
- If the navigation path is different (e.g. end users reachable directly from a contacts tab), adapt accordingly and document in status.md.
