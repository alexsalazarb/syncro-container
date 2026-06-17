# SE-12758 — WebView Auth Challenge Completion Handler Not Called

**Plan slug**: `SE-12758-webview-auth-challenge`
**Jira**: [SE-12758](https://syncrotech.atlassian.net/browse/SE-12758)
**Type**: Bug Fix
**Severity**: P3 — Low (1 event, 1 user, edge case)
**Status**: not-started
**Project**: syncro-flutter
**Branch convention**: `plan/SE-12758-webview-auth-challenge/task-{N}-{slug}`
**Base branch**: `develop`

---

## Problem

Production iOS crash (v1.6.1 build 433) in the embedded WebView login flow:

```
NSInternalInconsistencyException:
Completion handler passed to -[FWFNavigationDelegate
webView:didReceiveAuthenticationChallenge:completionHandler:]
was not called
```

**Crashlytics issue**: `0695473e5489f93da3cc9665ffb718a2`
**Signal**: REGRESSED — closed May 25, 2026 → regressed June 16, 2026 in v1.6.1

---

## Root Cause

`LoginWebView` (at `lib/features/authentication/login/presentation/login_web_view.dart`)
uses `webview_flutter` but does NOT register a handler for HTTP authentication
challenges (`onHttpAuthRequest`).

When iOS 26.5.0 sends an authentication challenge to `WKWebView` and the user
dismisses the screen before the challenge is resolved, the native
`FWFNavigationDelegate` holds a completion handler that never gets called →
iOS throws `NSInternalInconsistencyException`.

**Contributing factor (SE-12529 regression)**: commit `ac2d33ca` added
`_clearWebViewCache()` to `dispose()`. This runs async WebView operations after
disposal, creating a race condition with any pending native callbacks. iOS 26.5.0
appears stricter about enforcing the completion handler contract than earlier iOS
versions.

**Packages involved**:
- `webview_flutter 4.13.1`
- `webview_flutter_wkwebview 3.26.0`

---

## Fix Summary

Add `onHttpAuthRequest` to the `NavigationDelegate` in `login_web_view.dart`.
This registers a Dart-side handler so the native `FWFNavigationDelegate` always
has a completion handler path → completion handler is ALWAYS called, even if
the WebView is dismissed mid-challenge.

---

## Tasks

| # | Task | Status |
|---|------|--------|
| 01 | [Fix auth challenge handler](task-01-fix-auth-challenge-handler/task.md) | not-started |
| 02 | [Regression test](task-02-regression-test/task.md) | not-started |

---

## Success Criteria

- [ ] `onHttpAuthRequest` is registered in `LoginWebView`'s `NavigationDelegate`
- [ ] No `NSInternalInconsistencyException` when dismissing the WebView mid-challenge
- [ ] Widget test covers the `onHttpAuthRequest` code path
- [ ] Crash does not reappear in Crashlytics after deploy
- [ ] Verified manually on iOS 26 (iPhone or simulator)

---

## Affected Files

| File | Change |
|------|--------|
| `lib/features/authentication/login/presentation/login_web_view.dart` | Add `onHttpAuthRequest` handler to `NavigationDelegate` |
| `test/features/authentication/login/presentation/login_web_view_test.dart` | New — widget test for auth challenge handler |
