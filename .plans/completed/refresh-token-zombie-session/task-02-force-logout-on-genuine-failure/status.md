# Status: Force logout when a refresh is genuinely rejected by the server

**Current Status**: complete
**Last Updated**: 2026-09-09
**Agent**: implementer
**Branch**: plan/refresh-token-zombie-session/task-02-force-logout-on-genuine-failure
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-09-09 | not-started | — | Task created |
| 2026-09-09 | in-progress | implementer | Branched from task-01's tip; confirmed `AuthTokenClientFactory`/`AuthTokenClient` present |
| 2026-09-09 | complete | implementer | Forced logout wired in `refreshToken()`'s failure branch; new/updated tests pass; `flutter analyze` clean; full suite green except the pre-existing unrelated `chat_models_test.dart` failure |

## Blockers

None

## Artifacts

- `lib/features/authentication/login/infrastructure/login_repository_impl.dart` — modified: imports `AuthenticationCubit` and `dart:async`; in `refreshToken()`'s failure branch, `if (failure is UnauthorizedFailure)` calls `unawaited(GetIt.instance.get<AuthenticationCubit>().logOut())` before returning `Left(failure)`.
- `test/features/authentication/login/infrastructure/login_repository_impl_test.dart` — modified: added local `@GenerateMocks([AuthenticationCubit])` + `login_repository_impl_test.mocks.dart` (generated), registers `MockAuthenticationCubit` in GetIt for the `refreshToken` group, and asserts `logOut()` is called exactly once for the 401/`UnauthorizedFailure` case and never called for the success, "no token" guard, and transient-500 cases.
- `test/features/authentication/login/infrastructure/login_repository_impl_test.mocks.dart` — generated (new file, via `build_runner`).

## Adaptations

- **Local mock generation instead of shared file**: Initially tried adding `@GenerateMocks([AuthenticationCubit])` to the shared `test/features/general/repositories_impl_test.dart` (which backs the `MockNetworkServiceImpl`/`MockEnvironment`/etc. used elsewhere in this test). This caused `ambiguous_import` analyzer errors in three unrelated test files (`login_page_test.dart`, `passkey_enrollment_cubit_test.dart`, `passkey_flow_test.dart`, `passkey_login_button_test.dart`) that already generate their own local `MockAuthenticationCubit` via their own `@GenerateMocks` annotations. Reverted that change and instead added the annotation locally inside `login_repository_impl_test.dart` (matching the established per-file pattern already used by those other test files), producing its own `login_repository_impl_test.mocks.dart`. No production code or scope impact — test-infrastructure judgment call only.
- **Test coverage delivered via augmentation, not new standalone tests**: task.md's Testing section lists 3 "new test" bullets for the logout behavior. Rather than adding 3 separate new `test()` blocks, the required assertions (`logOut()` called for `UnauthorizedFailure`, not called for non-`UnauthorizedFailure`, not called for the "no token" guard) were added as additional `verify`/`verifyNever` calls inside the 4 existing `refreshToken` tests (success, no-token guard, 401/UnauthorizedFailure, transient-500), which already exercise exactly those 3 scenarios end-to-end (plus the success case). Avoids duplicating test scaffolding for scenarios already fully set up.
- Context's call-chain assumption (single change in `LoginRepositoryImpl.refreshToken()` covers both the reactive interceptor path and the cold-boot `AuthenticationCubit.checkAuthentication()` path) was verified against current code and holds. No changes were needed to `authentication_cubit.dart` or `rest_network_service.dart`.
