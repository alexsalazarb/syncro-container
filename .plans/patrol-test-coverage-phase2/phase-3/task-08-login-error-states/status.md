# Status

| Field | Value |
|-------|-------|
| **Status** | adapted |
| **Started** | 2026-06-19 |
| **Completed** | 2026-06-19 |
| **Branch** | `feature/testwork` |
| **PR** | N/A |

---

## Adaptation

**Login is WebView-based — wrong-credentials test not implementable via patrolWidgetTest.**

Investigation findings:
- `login_buttons.dart` → `_handleLogin()` calls `context.read<LoginCubit>().login()` which returns a URL, then pushes to `AppRoute.loginWebView` (a native `WebViewWidget` from `webview_flutter`).
- There are NO Flutter `TextField` widgets for email or password in the native widget tree. The credential form is server-rendered HTML inside the WebView.
- The error message after bad credentials is also rendered in the WebView's HTML — it is NOT a Flutter Snackbar or any other Flutter widget accessible to `patrolWidgetTest`.

`patrolWidgetTest` (from `patrol_finders`) can only interact with the Flutter widget tree. It cannot read or fill HTML inputs inside a `WebViewWidget`.

**What was done instead:**
- Added a top-of-file comment block to `authentication_test.dart` documenting this limitation and two recommended alternatives.
- Wrapped the existing test body in `withSocketFilter()` to align with the project test convention (was missing from this file).
- No new test case was added (impossible given the constraint above).

**Recommended alternatives for future coverage:**
1. Use full `patrol` (not `patrol_finders`) with native UI automation (UIAutomator2 / XCUITest) to drive the WebView HTML inputs directly.
2. Write a unit test for `LoginCubit` that mocks `LoginRepository` and asserts `LoginError` is emitted when the repository throws an auth failure.
