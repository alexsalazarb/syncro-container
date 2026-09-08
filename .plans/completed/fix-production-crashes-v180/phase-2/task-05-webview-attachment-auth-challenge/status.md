# Status: add `onHttpAuthRequest` to `AttachmentPreviewView`'s WebView (SE-12758 pattern, new call site)

**Current Status**: complete
**Last Updated**: 2026-09-07
**Agent**: claude
**Branch**: `plan/fix-production-crashes-v180` (syncro-flutter submodule, based on `develop`) — unified plan branch, see overview.md Branch Convention
**PR**: N/A (PR_INTEGRATION=false) — see plan overview for the current PR link

<!-- Status values: not-started | in-progress | complete | blocked | adapted -->

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-08-28 | not-started | claude | Task created as part of 1.7.1-crashlytics plan |
| 2026-09-07 | not-started | claude | Merged into fix-production-crashes-v180 (absorbed from the standalone 1.7.1-crashlytics plan, JIRA SE-13805 assigned) |
| 2026-09-07 | complete | claude | Fix + handler-logic test implemented, `flutter analyze` clean, full suite run (1 unrelated pre-existing failure, same as prior tasks) |

## Summary

Task's Context was exactly right (already verified once during the plan's Correction Log investigation): `attachment_preview_view.dart:44` had a bare `WebViewController()` with no `NavigationDelegate` at all. Added one with `onHttpAuthRequest: (request) => request.onCancel()`, mirroring `login_web_view.dart`'s SE-12758 fix exactly. Purely additive — no other navigation/loading behavior touched.

## Regression Test

`test/features/ticket/ticket_attachment/presentation/attachment_preview_view_test.dart` — mirrors `login_web_view_test.dart`'s approach: asserts the handler logic (`request.onCancel()` gets called) rather than asserting the real native delegate wiring, since neither WebView has a seam for that. Same testing constraint, same precedent, consistent with the sibling fix.

## Verification

- [x] `fvm flutter analyze` on changed files: clean, no issues
- [x] `fvm flutter test test/features/ticket/ticket_attachment/presentation/attachment_preview_view_test.dart`: 1/1 pass
- [x] Full suite (`fvm flutter test`): 2292 tests, 1 failure — same pre-existing `chat_models_test.dart` casing mismatch noted in prior tasks, unrelated, not touched

## Artifacts

- `lib/features/ticket/ticket_attachment/presentation/attachment_preview_view.dart` — fix
- `test/features/ticket/ticket_attachment/presentation/attachment_preview_view_test.dart` — new regression test
- Committed on the unified `plan/fix-production-crashes-v180` branch, pushed to `bla`

## Adaptations

None — task.md's plan matched reality exactly, including the reference test file to mirror.
