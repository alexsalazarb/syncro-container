# Task 02 — Regression Test

**Plan**: SE-12758-webview-auth-challenge
**Status**: not-started
**Depends on**: task-01 (must be complete first)
**Branch**: `plan/SE-12758-webview-auth-challenge/task-02-regression-test`
**Base**: `plan/SE-12758-webview-auth-challenge/task-01-fix-auth-challenge-handler`

---

## Objective

Add a widget test that covers the `onHttpAuthRequest` handler code path in
`LoginWebView`, confirming the handler is registered and callable.

---

## File Ownership

**Create**:
- `test/features/authentication/login/presentation/login_web_view_test.dart`

**Do NOT Modify**:
- `lib/` files — task-01 already handled the fix

---

## Test Strategy

The crash is triggered by a native iOS callback that can't be replicated in
unit/widget tests. The goal is not to reproduce the crash, but to assert
the handler registration exists and is functional.

### What to test

1. `LoginWebView` renders without exceptions
2. The `NavigationDelegate` receives an `onHttpAuthRequest` that calls `onCancel()`
   (verify via the mock callback path)

### Implementation approach

Use `webview_flutter`'s test helpers (`WebViewPlatform` fake) to set up a mock
platform, build `LoginWebView`, and assert the delegate was configured with an
`onHttpAuthRequest` callback.

If the webview platform mock is too complex, document a manual test case instead
and skip the widget test — correctness over coverage theatre.

---

## Manual Test Case (always document regardless of automated test)

```
Precondition: Device running iOS 26 (real device preferred; simulator acceptable)

Steps:
1. Open the app on the login screen
2. Tap the OAuth login button — WebView opens
3. Wait for the WebView to begin loading (~1-2 seconds)
4. Immediately press the back button / swipe back to dismiss the WebView
5. Repeat 3-5 times to exercise different timing windows

Expected result:
- App does not crash
- Login screen reappears normally
- No Crashlytics event for NSInternalInconsistencyException

Before the fix: crash occurs intermittently (depends on whether the
authentication challenge arrives exactly during WebView dismissal)
```

---

## Commit Message

```
SE-12758: add widget test for LoginWebView onHttpAuthRequest handler
```
