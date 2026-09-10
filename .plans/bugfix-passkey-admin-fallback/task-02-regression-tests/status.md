# Status: Rewrite/add passkey login tests for the admin-host fallback

**Current Status**: complete
**Last Updated**: 2026-09-10
**Agent**: Claude (execute-task)
**Branch**: `plan/bugfix-passkey-admin-fallback/task-02-regression-tests`
**PR**: N/A (PR_INTEGRATION=false)

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-09-10 | not-started | Task created |
| 2026-09-10 | in-progress | task-01 confirmed complete; branched off task-01's branch directly (not `develop`) since task-01 isn't merged yet and this task's tests need its fix in place |
| 2026-09-10 | complete | Both flagged tests rewritten, 1 new test added, 1 new repository-level test added. `flutter analyze` clean, `pre-commit-check` clean, full passkey suite green (109 tests) |

## Blockers

None.

## Artifacts

- `test/features/authentication/passkey/application/passkey_login_cubit_test.dart` — rewrote the empty-subdomain "no passkey" test to go through the real `getLoginChallenge()`/native-sign flow instead of asserting `verifyNever`; added a new test proving the admin-host fallback can succeed end-to-end
- `test/features/authentication/passkey/infrastructure/passkey_repository_impl_test.dart` — rewrote the "does NOT attempt to initialize NetworkService" test to assert it now initializes against `subdomainUrl('admin')`

## Adaptations

None beyond what task-01's status.md already flagged forward (the second pre-existing test at the repository level, incorporated into this task's scope as planned).
