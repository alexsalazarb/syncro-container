# Investigation — SE-12758

## Crashlytics Data

- **App**: Syncro Mobile (com.servably.syncro.mobile) — Production iOS
- **Version**: 1.6.1 (build 433)
- **Event time**: 2026-06-16T15:05:47Z
- **Device**: iPhone 16 Pro (iPhone17,1) — arm64e
- **iOS**: 26.5.0
- **App state at crash**: Background
- **Impacted**: 1 user, 1 session
- **Signal**: REGRESSED (closed 2026-05-25, reopened 2026-06-16 in v1.6.1)

## Exception

```
NSInternalInconsistencyException:
Completion handler passed to -[FWFNavigationDelegate
webView:didReceiveAuthenticationChallenge:completionHandler:]
was not called
```

Stack frames:
```
__exceptionPreprocess
objc_exception_throw
_call_dispose_helpers_excp
_Block_release
__destroy_helper_block_e8_32s40s (FBLPromise.m)
_call_dispose_helpers_excp
_Block_release
```

## Origin Analysis

### Regression source — commit `ac2d33ca` (SE-12529, May 28, 2026)

SE-12529 modified `login_web_view.dart` to:
1. Add `WebViewCookieManager().clearCookies()` as the first step of `_clearWebViewCache()`
2. Move URL loading to AFTER cookie clear via `.then((_) { _controller.loadRequest(...) })`
3. Call `_clearWebViewCache()` in `dispose()` (unawaited async)

The `dispose()` async operations create a window where async WebView operations
run concurrently with the widget being unmounted. If the WKWebView receives an
auth challenge during this window, the native completion handler is allocated but
never resolved (Dart side has no handler), causing the crash on iOS 26.5.0.

### Missing handler (pre-existing gap)

`NavigationDelegate` in `login_web_view.dart` does NOT set `onHttpAuthRequest`.
Without it, `FWFNavigationDelegate` (the native iOS delegate from
`webview_flutter_wkwebview`) has no Dart callback to invoke when iOS calls
`webView:didReceiveAuthenticationChallenge:completionHandler:`. If the block
containing the handler is deallocated (e.g. WebView dismissed), iOS enforces
the must-call contract by throwing.

iOS 26.5.0 appears to be stricter about this contract than iOS 17.x.

## Why Existing Tests Missed This

No widget test exists for `LoginWebView`. The bug is triggered by a native iOS
callback + timing condition during WebView disposal — not exercisable via
unit tests alone without mocking the native layer.

## Feature History

- `login_web_view.dart` introduced in initial WebView login implementation
- SE-12529 (May 28, 2026) — last modification, introduced dispose race condition
- Issue first seen in v1.2.1, closed May 25, 2026, regressed in v1.6.1

## Cross-Project Assessment

Single-project bug. Only affects `syncro-flutter` iOS.

## Confidence: High

The missing `onHttpAuthRequest` handler is the direct cause. `FWFNavigationDelegate`'s
`webView:didReceiveAuthenticationChallenge:completionHandler:` method in
`webview_flutter_wkwebview 3.26.0` requires a Dart-side handler to be registered
to guarantee the completion handler is called. The SE-12529 dispose change
created the timing window for the crash to occur.
