# Task: add `onHttpAuthRequest` to `AttachmentPreviewView`'s WebView (SE-12758 pattern, new call site)

**Plan**: Fix Production Crashes — v1.8.0 (fix-production-crashes-v180)
**Phase**: 2
**Task ID**: task-05
**Task Path**: `phase-2/task-05-webview-attachment-auth-challenge`
**Depends On**: None
**JIRA**: [SE-13805](https://syncrotech.atlassian.net/browse/SE-13805) — consider linking to SE-12758 as a "relates to" for traceability
**Crashlytics Issue**: `b0914a639eb5e4b996dec3e8f1147a4b` ([console](https://console.firebase.google.com/v1/appid/project/syncromsp-ios/crashlytics/app/1:920223298498:ios:a0d84c75923c83b35b5c5b/issues/b0914a639eb5e4b996dec3e8f1147a4b)) — FATAL, firstSeen/lastSeen 1.7.0

## Objective

Fix `NSInternalInconsistencyException: Completion handler passed to -[webview_flutter_wkwebview.NavigationDelegateImpl webView:didReceiveAuthenticationChallenge:completionHandler:] was not called`, occurring in `AttachmentPreviewView`.

## Context

**This is the same crash class SE-12758 already fixed once (2026-06-17)** — but at a different call site. SE-12758's issue ID was `0695473e5489f93da3cc9665ffb718a2` in `LoginWebView`, delegate class `FWFNavigationDelegate`. This new issue (`b0914a639...`) has a different issue ID and a different delegate class name (`NavigationDelegateImpl`, likely reflecting a `webview_flutter_wkwebview` package version change since then — `pubspec.yaml` currently pins `webview_flutter_wkwebview: ^3.26.0` with the comment `# pin levantado: bug login fixeado en 3.18.3`, confirming the SE-12758 fix context).

Verified during planning: this app has **two** `webview_flutter` consumers:
- `lib/features/authentication/login/presentation/login_web_view.dart` — has `onHttpAuthRequest` registered (line 114, the SE-12758 fix). **Out of scope — already fixed, do not modify.**
- `lib/features/ticket/ticket_attachment/presentation/attachment_preview_view.dart` (line 44) — `controller: WebViewController()` with **no `NavigationDelegate` at all**, confirmed via grep during planning (no `onHttpAuthRequest` match in this file).

This is a strong, code-confirmed match: `AttachmentPreviewView` is the un-patched second call site. Same root cause as SE-12758 — no Dart-side `onHttpAuthRequest` handler means the native `NavigationDelegateImpl` has no completion path if the WebView is dismissed mid-challenge, and iOS throws `NSInternalInconsistencyException`.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase bla develop`
- [ ] Create the task branch: `git switch -c plan/fix-production-crashes-v180/phase-2/task-05-webview-attachment-auth-challenge`
- [ ] Read `.plans/completed/SE-12758-webview-auth-challenge/overview.md` and `task-01-fix-auth-challenge-handler/task.md` for the exact fix pattern to mirror
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/ticket/ticket_attachment/presentation/attachment_preview_view.dart` | modify | Add a `NavigationDelegate` with `onHttpAuthRequest` to the `WebViewController`, mirroring `login_web_view.dart`'s pattern (line 114, uses `request.onCancel()`) |
| `syncro-flutter/test/features/ticket/ticket_attachment/presentation/attachment_preview_view_test.dart` | create | No test file exists for this widget yet (confirmed via `fd` during planning); mirror `test/features/authentication/login/presentation/login_web_view_test.dart`'s structure for the `onHttpAuthRequest` coverage |

### Do NOT Modify

- `syncro-flutter/lib/features/authentication/login/presentation/login_web_view.dart` — already fixed by SE-12758, out of scope
- `syncro-flutter/test/features/authentication/login/presentation/login_web_view_test.dart` — belongs to the file above, read-only reference

## Implementation Steps

### Step 1: Register the NavigationDelegate
In `attachment_preview_view.dart`, change the bare `WebViewController()` at line 44 to set a `NavigationDelegate` (via `.setNavigationDelegate(...)`) with an `onHttpAuthRequest` handler that calls `request.onCancel()`, identical in structure to `login_web_view.dart` line 114.

### Step 2: Confirm no other behavior regresses
`AttachmentPreviewView` likely doesn't hit HTTP auth challenges in normal use (it's previewing ticket attachments, not an OAuth login flow) — confirm the change is purely additive and doesn't alter existing navigation/loading behavior for attachment URLs.

## Testing

- [ ] New widget test: `onHttpAuthRequest` is registered on `AttachmentPreviewView`'s `NavigationDelegate` and calls `request.onCancel()` when invoked — mirror `login_web_view_test.dart`'s test structure and mocking approach.
- [ ] Existing attachment preview behavior (loading/displaying an attachment URL) still works.
- [ ] `fvm flutter analyze` passes
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No new KB doc needed — this is the same documented pattern from SE-12758, just applied to a second call site. If a KB doc doesn't already generalize "always register `onHttpAuthRequest` on any new `webview_flutter` `WebViewController`" as a project convention, consider adding one line to `docs/kb-projects/syncro-flutter/engineering/best-practices/flutter-standards.md` so a third call site doesn't repeat this — optional, not blocking.

## Completion Criteria

- [ ] `onHttpAuthRequest` registered in `AttachmentPreviewView`'s `NavigationDelegate`
- [ ] New widget test passes
- [ ] No new occurrence of this issue in Crashlytics after the fix ships (follow-up check, not blocking this task's completion)
