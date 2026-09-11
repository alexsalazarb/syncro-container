# Status: Defect — false-positive enrollment clear on admin-host fallback

**Current Status**: complete
**Last Updated**: 2026-09-11
**Agent**: Claude (execute-task)
**Branch**: `plan/bugfix-passkey-admin-fallback/task-03-defect-false-clear-on-admin-fallback`
**PR**: N/A (PR_INTEGRATION=false)

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-09-11 | not-started | Defect task created — found during manual staging verification of task-01/task-02 |
| 2026-09-11 | complete | Guarded the clear behind `AppSharedPreferences.getSubdomain().isNotEmpty`; added regression test proving flags survive `credential_not_recognized` via the admin-fallback path while the known-tenant case still clears as before. `flutter analyze` clean, `pre-commit-check` clean, 110/110 passkey tests green |

## Blockers

None — can be implemented independently of Justin's backend fix; it's a client-side safety guard regardless of whether/when the backend tenant-resolution gap is fixed.

## Artifacts

- `lib/features/authentication/passkey/application/passkey_login_cubit.dart` — `credential_not_recognized` only clears local enrollment flags when `AppSharedPreferences.getSubdomain().isNotEmpty`
- `test/features/authentication/passkey/application/passkey_login_cubit_test.dart` — new test: admin-fallback path does not clear flags on `credential_not_recognized`

## Adaptations

None
