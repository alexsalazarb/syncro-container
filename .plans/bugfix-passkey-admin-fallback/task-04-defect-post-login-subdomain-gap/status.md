# Status: Defect — passkey login never learns the real subdomain, breaking the post-login `/me` call

**Current Status**: complete
**Last Updated**: 2026-09-14
**Agent**: Claude (execute-task)
**Branch**: `plan/bugfix-passkey-admin-fallback/task-04-defect-post-login-subdomain-gap` (local only, not yet pushed — see Adaptations)
**PR**: N/A (PR_INTEGRATION=false)

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-09-14 | not-started | Defect task created — found during real-device staging verification on `ss1`, right after backend MR 19414's global-credential-lookup fix (commit `ac8ce8`) made passkey sign-in itself start succeeding. Backend has already shipped its half of the fix (commit `07477a6d`, `subdomain` field on the login-verify response, deployed and confirmed live on `ss1`) — this task is entirely client-side. |
| 2026-09-14 | in-progress | Cloned `syncro-flutter` into the container at its configured location (was missing locally — gitignored, `fd` didn't surface the existing checkout at first; confirmed it already existed on `develop`, tasks 1-3's fix already merged). Branched off `develop`. |
| 2026-09-14 | complete | Resolved the Step 3 refresh-callback design question: reused `GetIt.instance.get<LoginRepository>().refreshToken()` (already a registered singleton, same pattern this class already uses for `TokenCubit`) instead of duplicating `AuthTokenClient`/refresh-grant logic. `flutter analyze` clean (both scoped and full project), `pre-commit-check` clean, 128/128 passkey-area tests + 51/51 login-area tests green (including 3 new `verifyLogin` tests and 1 new deserializer test). |

## Blockers

None — backend prerequisite (MR 19414 commit `07477a6d`) is already deployed and confirmed live on `ss1`.

## Artifacts

- `lib/features/authentication/login/domain/get_token_response.dart` — added optional `subdomain` field to `GetTokenResponse`
- `lib/features/authentication/login/infrastructure/get_token_deserializer.dart` — parses `data['subdomain']`
- `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` — `verifyLogin()` now uses the backend-provided subdomain for `TokenData`, persists it via `AppSharedPreferences.setSubdomain()`, and re-points `networkService` to the real tenant host with a working refresh-token callback (`LoginRepository.refreshToken()` via GetIt) before returning, when the backend response carries one
- `test/features/authentication/login/infrastructure/get_token_deserializer_test.dart` — subdomain parsing coverage
- `test/features/authentication/passkey/infrastructure/passkey_repository_impl_test.dart` — 3 new tests: backend subdomain wins over locally-persisted one, `networkService` re-init + refresh-callback wiring, and the older-backend (no `subdomain`) fallback path

## Adaptations

Did not push the task branch to `origin`/`bla`, and did not squash into `develop` — this plan's established convention (tasks 1-3) is to batch multiple tasks into a single local `develop` commit and decide the push timing deliberately, not push/merge per task automatically. Left as a local commit on the task branch pending Alex's call on when to fold it into `develop`.
