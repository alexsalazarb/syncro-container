# Task 01 — Fix Auth Challenge Handler

**Plan**: SE-12758-webview-auth-challenge
**Status**: not-started
**Branch**: `plan/SE-12758-webview-auth-challenge/task-01-fix-auth-challenge-handler`
**Base**: `develop`

---

## Objective

Register an `onHttpAuthRequest` handler in `LoginWebView`'s `NavigationDelegate`
so the native `FWFNavigationDelegate` always has a completion handler path.
This prevents `NSInternalInconsistencyException` when the WebView is dismissed
while iOS is processing an authentication challenge.

---

## File Ownership

**Modify**:
- `lib/features/authentication/login/presentation/login_web_view.dart`

**Do NOT Modify**:
- Any other file in this task

---

## Implementation Steps

### 1. Add `onHttpAuthRequest` to the existing `NavigationDelegate`

In `_LoginWebViewState.initState()`, the `NavigationDelegate` is built starting
at line ~54. Add the `onHttpAuthRequest` parameter:

```dart
final navigationDelegate = NavigationDelegate(
  onProgress: (int progress) { ... },
  onPageStarted: (String url) { ... },
  onPageFinished: (String urlweb) async { ... },
  onWebResourceError: (WebResourceError error) { ... },
  onNavigationRequest: (NavigationRequest request) { ... },
  // ADD THIS:
  onHttpAuthRequest: (HttpAuthRequest request) {
    request.onCancel();
  },
);
```

`request.onCancel()` tells the WKWebView to cancel the auth challenge. This is
the correct default for a login WebView that uses OAuth redirect — server-side
authentication challenges (like NTLM, Kerberos, or basic auth on sub-resources)
should be cancelled to avoid blocking the flow. The completion handler is ALWAYS
called, which prevents the crash.

> If the login server requires HTTP Basic/Digest auth on the challenge URL itself,
> use `request.onProceed(WebViewCredential(...))` with the appropriate credentials
> instead. For the current OAuth redirect flow, `onCancel` is the right choice.

### 2. No other changes needed

The `dispose()` async race is a contributing factor but not the primary fix.
Changing `dispose()` could break the SE-12529 fix (cookie clearing). The
`onHttpAuthRequest` handler is sufficient: once registered, the completion handler
path is always available regardless of WebView lifecycle state.

---

## Verification

After implementing, verify:

1. `fvm flutter analyze` — no new warnings
2. `fvm dart format lib/features/authentication/login/presentation/login_web_view.dart`
3. Build and test the OAuth login flow on a real iOS device (or simulator)
4. Verify no crash when pressing back during the WebView login

---

## Commit Message

```
SE-12758: register onHttpAuthRequest handler in LoginWebView to prevent iOS completion handler crash
```
