# Status: Isolate the OAuth refresh call from the shared 401-retry interceptor

**Current Status**: adapted
**Last Updated**: 2026-09-09
**Agent**: Claude (implementer)
**Branch**: plan/refresh-token-zombie-session/task-01-isolate-refresh-network-call
**PR**: N/A

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-09-09 | not-started | — | Task created |
| 2026-09-09 | in-progress | Claude (implementer) | Starting implementation |
| 2026-09-09 | adapted | Claude (implementer) | Implementation complete, all tests pass; branched from `develop` instead of `main` (see Adaptations) |

## Blockers

None

## Artifacts

- `lib/core/networking/services/rest_api/auth_token_client.dart` (new) — dedicated, minimal Dio client for `POST /oauth/token` refresh-grant, no 401-retry interceptor. Maps HTTP 400/401 -> `UnauthorizedFailure`; other failures (network error, 5xx) -> `MessageFailure`/`UnexpectedFailure`.
- `lib/features/authentication/login/infrastructure/login_repository_impl.dart` (modified) — `refreshToken()` now builds an `AuthTokenClient` via an injectable `AuthTokenClientFactory` (constructor param, defaults to `AuthTokenClient.new`) instead of calling `networkService.sendRequestCall(...)`. `getToken()`, `me()`, `oauthLogin()`, `forgotPassword()`, `_initNetworkService()` untouched.
- `test/core/networking/services/rest_api/auth_token_client_test.dart` (new) — 5 tests: 401 (bounded-time proof), 400, success mapping, transient 500, connection error. Uses a hand-written fake `HttpClientAdapter` (no new test dependency needed).
- `test/features/authentication/login/infrastructure/login_repository_impl_test.dart` (modified) — `refreshToken` group rewired to inject `authTokenClientFactory` (built on the same fake `HttpClientAdapter`) instead of mocking `networkService.sendRequestCall`. Old "network fails" test replaced by two new ones: 401-deadlock-proof (bounded `.timeout()`, asserts `Left(UnauthorizedFailure)`) and 500-transient (asserts `Left(non-UnauthorizedFailure)`). Success-path test still asserts `TokenCubit.setToken`/`storageManager.saveToken` are called with the new token.

## Adaptations

1. **Base branch: `develop`, not `main`.** Task.md's "Before You Start" says `git switch main && git pull --rebase origin main`. I verified this is stale for this repo: `main` is missing ~1300 lines of code across the authentication `login/infrastructure` directory that this task depends on (e.g. the current `TokenCubit`/`GetIt`-based `refreshToken()` this task modifies doesn't exist on `main` at all). Cross-checked against the most recently completed plan (`fix-production-crashes-v180`): its task branch's merge-base is a commit that exists on `develop` but not on `main`, confirming task branches in this repo are conventionally cut from `develop`. Branched from `develop` instead; recommend the plan template/AGENTS.md be corrected in a follow-up (out of scope here).
2. **`LoginRepositoryImpl` constructor gained one new optional parameter** (`AuthTokenClientFactory? authTokenClientFactory`, defaulting to `AuthTokenClient.new`) — not explicitly called for by task.md, but necessary to make `refreshToken()` testable without a real network call (mirrors how `rest_network_service_5xx_test.dart` already documents that `RestNetworkServiceImpl` has no such seam and can't be unit-tested for its Dio-level behavior — I didn't want to repeat that gap for the new client). Default value preserves existing behavior for the one production call site (`service_locator.dart`), which passes no such argument.
3. **Did not extract a shared failure-mapping helper from `RestNetworkServiceImpl._handleFailure`.** Per task.md step 1's explicit "consider extraction vs. duplication" prompt: the OAuth token endpoint's error shape (`{"error": ..., "error_description": ...}`) and rules (400/401 -> `UnauthorizedFailure`) are narrow and specific to this endpoint, while `_handleFailure`'s reusable logic (422 validation-body parsing, 500/501 Crashlytics reporting, generic `ErrorModel` mapping) doesn't apply to this endpoint at all. Wrote a small, self-contained `_mapFailure`/`_extractOAuthErrorMessage` pair directly in `AuthTokenClient` instead, with a comment pointing back to this task.
4. **Reused `AppRequests.getToken.requestOption().operationNameOrPath`** for the `/oauth/token` path inside `AuthTokenClient` rather than hardcoding the string a second time, so there's one source of truth for the endpoint path.
5. **KB gap noted, not fixed (per task.md's explicit instruction):** the Dio compact KB doc (`.ai-framework/knowledge-base/flutter/integrations/dio-patterns.compact.md`) documents the `Completer`-coalescing pattern but doesn't call out the self-recursion deadlock gotcha this task fixes. Task.md explicitly says not to update `docs/kb-projects/syncro-flutter/technical/integrations/oauth.md`/`ai-patterns/known-issues.md` as part of this task (out of scope, follow-up). Not updated.
