# Task: Force logout when a refresh is genuinely rejected by the server

**Plan**: refresh-token-zombie-session
**Phase**: None (flat plan)
**Task ID**: task-02
**Task Path**: task-02-force-logout-on-genuine-failure
**Depends On**: task-01-isolate-refresh-network-call
**JIRA**: [SE-13836](https://syncrotech.atlassian.net/browse/SE-13836)

## Objective

When `LoginRepositoryImpl.refreshToken()` determines the refresh was genuinely rejected by the server (`UnauthorizedFailure`, from task-01's dedicated client) — as opposed to a transient failure or "no token present" — force an automatic logout, so the user is redirected to login instead of being left in a broken "still logged in but nothing loads" state.

## Context

`AuthenticationCubit` is a GetIt singleton (`lib/app/dependency/service_locator.dart:54-65`, registered as `getIt.registerSingleton(AuthenticationCubit(...))`), explicitly documented in a comment there as global "because AuthenticationCubit ... and ChatWebSocketService depend on it" — it's reachable from anywhere via `GetIt.instance.get<AuthenticationCubit>()`, not just from widgets with a `BuildContext`.

`LoginRepositoryImpl` (`lib/features/authentication/login/infrastructure/login_repository_impl.dart`) already reaches into GetIt singletons directly from repository code — see `GetIt.instance.get<TokenCubit>()` at lines ~117 and ~146 inside `refreshToken()`. Reaching `AuthenticationCubit` the same way from the same method is consistent with that existing pattern, not a new layering violation.

Because BOTH call sites that trigger a refresh — the reactive Dio 401-interceptor (via the closure built in `_initNetworkService()`: `() async => (await refreshToken()).fold((_) => null, (token) => token)`) and the cold-boot path (`AuthenticationCubit._refreshToken()`, which calls `loginRepository.refreshToken()` directly) — ultimately call the SAME `LoginRepositoryImpl.refreshToken()` method, a single change inside that method's failure branch covers both paths. You should NOT need to touch `authentication_cubit.dart` or `rest_network_service.dart` for this task — if you find yourself needing to, stop and reconsider, since that likely means the single-change approach doesn't hold and the plan's task split needs revisiting (note it in `status.md` Adaptations either way).

`AuthenticationCubit.logOut()` (`lib/features/authentication/application/authentication_cubit.dart:192-199`) already does the right cleanup: `NotificationManager.instance.logout()`, `logoutUseCase.call()` (clears persisted tokens via `storageManager`), and `_handleUnauthenticated()` (emits `AuthenticationUnauthenticated()`). Use this existing method — do not duplicate its cleanup logic inline.

**Scope boundary — read carefully before implementing:**
- Genuine rejection (`UnauthorizedFailure`) → force `logOut()`.
- The existing early guard in `refreshToken()` for "no refresh token available" (`tokenData == null || tokenData.refreshToken.isEmpty`) → leave as a plain `Left(MessageFailure(...))`, do NOT force logout there. In practice this guard is rarely reached mid-session (the app wouldn't have a valid session without a token in the first place), and forcing a full `logoutUseCase.call()` when there was never a real session to clean up is unnecessary churn — not part of this fix's scope.
- Any other failure (network error, timeout, 5xx, unexpected exception) → leave as-is, do NOT force logout. Only a confirmed server-side rejection should log the user out; transient failures should be allowed to fail silently and retried on the next request, as today.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase origin develop` (NOT `main` — `main` is vestigial for this project)
- [ ] If SE-13836 has updates in its description/comments since plan creation, check for latest requirements
- [ ] Verify task-01-isolate-refresh-network-call is `complete` in its `status.md` — this task depends on the `UnauthorizedFailure` signal it introduces
- [ ] Read `lib/features/authentication/login/infrastructure/login_repository_impl.dart` (specifically `refreshToken()`, post task-01's changes)
- [ ] Read `lib/features/authentication/application/authentication_cubit.dart` (`logOut()`, `_refreshToken()`, `_getUser()`) to confirm the call chain assumption above still holds after task-01's changes
- [ ] Read existing tests: `test/features/authentication/login/infrastructure/login_repository_impl_test.dart` and `test/features/authentication/application/authentication_cubit_test.dart`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/authentication/login/infrastructure/login_repository_impl.dart` | modify | Single addition in `refreshToken()`'s failure branch: when `failure is UnauthorizedFailure`, call `GetIt.instance.get<AuthenticationCubit>().logOut()` before returning `Left(failure)`. |
| `test/features/authentication/login/infrastructure/login_repository_impl_test.dart` | modify | Add tests for the forced-logout behavior and its absence on non-`UnauthorizedFailure` cases |

### Do NOT Modify

- `lib/core/networking/services/rest_api/rest_network_service.dart` — no change needed here; the interceptor still just does `handler.next(e)` on a null token, same as before
- `lib/features/authentication/application/authentication_cubit.dart` — no change needed if the single-change approach in Context holds; if it doesn't, stop and document why in `status.md` rather than expanding scope unilaterally
- Everything task-01 already owns/changed

## Implementation Steps

### Step 1: Detect genuine rejection

In `LoginRepositoryImpl.refreshToken()`, in the `response.fold` failure branch (currently just `logger('ERROR=> $failure'); return Left(failure);`), add a check: if `failure is UnauthorizedFailure`, call `GetIt.instance.get<AuthenticationCubit>().logOut()` (fire-and-forget is fine — `logOut()` is `Future<void>`, but `refreshToken()`'s caller doesn't need to wait on the logout side effect to get its own `Left(failure)` back; use `unawaited()` per this project's convention for fire-and-forget service calls if that fits, or await it if there's a clear reason to — use your judgement, but don't block the refresh's own return on it unnecessarily).

### Step 2: Confirm both call sites benefit automatically

- Trace the reactive path: `RestNetworkServiceImpl._handleError` → `_getOrCreateRefreshToken()` → the closure from `_initNetworkService()` → `LoginRepositoryImpl.refreshToken()`. Confirm the forced logout fires here without any change to `rest_network_service.dart`.
- Trace the cold-boot path: `AuthenticationCubit.checkAuthentication()` → `_getUser()` → `_refreshToken()` → `loginRepository.refreshToken()`. Confirm the forced logout fires here too, and that it composes sanely with `_getUser()`'s own subsequent `_handleUnauthenticated()` call (both end up emitting `AuthenticationUnauthenticated()` — that's redundant but harmless; do not try to suppress the redundancy unless it causes an actual test failure or visible glitch).

## Testing

- [ ] New test: `refreshToken()` returning `UnauthorizedFailure` results in `AuthenticationCubit.logOut()` being invoked (verify via mock/spy on the GetIt-registered `AuthenticationCubit`, matching whatever DI-mocking convention `login_repository_impl_test.dart` already uses for `TokenCubit`)
- [ ] New test: `refreshToken()` returning a non-`UnauthorizedFailure` (e.g. `MessageFailure` from a transient error) does NOT trigger `logOut()`
- [ ] New test: the "no refresh token available" early-guard path does NOT trigger `logOut()`
- [ ] Existing `authentication_cubit_test.dart` tests for `checkAuthentication()`/`_getUser()`/`logOut()` still pass unmodified
- [ ] Existing tests still pass
- [ ] Linter / static analysis passes: `fvm flutter analyze`
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Do NOT update the stale `oauth.md`/`known-issues.md` KB docs as part of this task — explicit follow-up outside this plan's scope (see overview.md Scope section)
- [ ] No new KB doc required for this narrowly-scoped fix

## Completion Criteria

- [ ] A genuinely-rejected refresh (`UnauthorizedFailure`) forces `AuthenticationCubit.logOut()`, from both the reactive and cold-boot paths
- [ ] Transient failures and "no token" do NOT force logout
- [ ] All tests pass
- [ ] No regressions in existing functionality
- [ ] Documentation / KB updates completed or explicitly marked not needed (see above — explicitly not needed)
- [ ] Changes committed to `plan/refresh-token-zombie-session/task-02-force-logout-on-genuine-failure` branch
- [ ] Status updated in `status.md`
