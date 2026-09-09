# Task: Isolate the OAuth refresh call from the shared 401-retry interceptor

**Plan**: refresh-token-zombie-session
**Phase**: None (flat plan)
**Task ID**: task-01
**Task Path**: task-01-isolate-refresh-network-call
**Depends On**: None
**JIRA**: [SE-13836](https://syncrotech.atlassian.net/browse/SE-13836)

## Objective

Make `LoginRepositoryImpl.refreshToken()` issue its `POST /oauth/token` (`grant_type=refresh_token`) request through a dedicated, minimal Dio client that does NOT go through `RestNetworkServiceImpl`'s 401-retry interceptor — so the refresh call can never re-enter its own retry logic and deadlock the shared `_refreshTokenCompleter`.

## Context

Full root cause is documented in `../overview.md` and in engram (`bugs/refresh-token-zombie-session`, project `syncro-flutter`). Summary of what matters for this task:

- `RestNetworkServiceImpl` (`lib/core/networking/services/rest_api/rest_network_service.dart`) is ONE long-lived instance for the entire app session. Its `_handleError` interceptor (lines ~87-111) intercepts any 401 whose request URI matches the Dio instance's `baseUrl`, and calls `_getOrCreateRefreshToken()` (lines ~113-147), which uses a `Completer<String?>` (`_refreshTokenCompleter`) so concurrent 401s share a single in-flight refresh.
- `LoginRepositoryImpl.refreshToken()` (`lib/features/authentication/login/infrastructure/login_repository_impl.dart:116-155`) currently calls `networkService.sendRequestCall(request: AppRequests.getToken.requestOption(), ...)` — i.e. it goes through the SAME `NetworkService` singleton / same Dio instance / same interceptors as every feature request, because `AppRequests.getToken` resolves to the relative path `/oauth/token` (`lib/core/networking/commons/app_requests_auth.dart:12`), which Dio resolves against the same `baseUrl`.
- Because of that, if the refresh-grant request itself comes back 401 (real OAuth `invalid_grant` behavior on some servers, or because `_setHeaders` — lines ~149-161 — slapped the current *expired* access token onto the refresh request as `Authorization: Bearer`), `_handleError` fires again for that nested request while `_refreshTokenCompleter` is still non-null (the original attempt hasn't resolved) — so it awaits the SAME completer's future via line ~116 (`return _refreshTokenCompleter!.future;`). Nothing will ever call `.complete()` on it, because the code that would is the very call blocked awaiting it. This is the deadlock. Once stuck, it's stuck for the rest of the app's life — every future 401, on any feature, hangs forever.
- `LoginRepositoryImpl.refreshToken()` reads the current token from `TokenCubit` (GetIt singleton, `GetIt.instance.get<TokenCubit>().state`) and, on success, writes the new token back to `TokenCubit` AND `storageManager.saveToken(...)`. This logic itself is correct and should NOT change — only the transport (how the HTTP call is made) changes.
- The Dio compact KB doc (`.ai-framework/knowledge-base/flutter/integrations/dio-patterns.compact.md`) documents the `Completer`-coalescing pattern as correct and states "Never have unbounded retry loops — refresh once and fail if it doesn't work" — it does not call out this specific self-recursion gotcha. Note it as a KB gap (see Documentation section below) rather than treating the existing doc as wrong.

**Do NOT touch** `LoginRepositoryImpl.getToken()` (the initial authorization-code exchange during login). It goes through the same shared `RestNetworkServiceImpl`/`_initNetworkService()` path today, but it cannot deadlock: at that point in the flow `TokenCubit.state` is still `null` (no session yet), so even if `getToken()` itself 401s, `_getOrCreateRefreshToken()`'s refresh attempt short-circuits immediately at `LoginRepositoryImpl.refreshToken()`'s guard (`if (tokenData == null || tokenData.refreshToken.isEmpty) return Left(...)`) without making a second network call. There is nothing to fix there — leave it as-is.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase origin develop` (NOT `main` — `main` is vestigial for this project)
- [ ] If SE-13836 has updates in its description/comments since plan creation, check for latest requirements
- [ ] Read `lib/core/networking/services/rest_api/rest_network_service.dart` in full (already read during investigation — re-confirm line numbers haven't shifted)
- [ ] Read `lib/features/authentication/login/infrastructure/login_repository_impl.dart` in full
- [ ] Read existing tests: `test/core/networking/services/rest_api/rest_network_service_5xx_test.dart` and `test/features/authentication/login/infrastructure/login_repository_impl_test.dart` — match their mocking style (they define the project's Dio/NetworkService test-double conventions; don't invent a new mocking approach)
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/core/networking/services/rest_api/auth_token_client.dart` (name is a suggestion — pick whatever fits existing naming) | create | Minimal Dio-based client, dedicated to `POST /oauth/token`. No 401-retry interceptor. |
| `lib/features/authentication/login/infrastructure/login_repository_impl.dart` | modify | Only `refreshToken()` changes to use the new client instead of `networkService.sendRequestCall`. `getToken()`, `me()`, `oauthLogin()`, `forgotPassword()`, `_initNetworkService()` are untouched by this task (task-02 makes one more small change to `refreshToken()` — read task-02 before finishing this one so you don't have to re-review the method twice, but do NOT implement task-02's change here). |
| `test/features/authentication/login/infrastructure/login_repository_impl_test.dart` | modify | Add/adjust tests for `refreshToken()` using the new client |
| New test file for the dedicated client (e.g. `test/core/networking/services/rest_api/auth_token_client_test.dart`) | create | Unit tests for the client itself |

### Do NOT Modify

- `lib/features/authentication/application/authentication_cubit.dart` — owned by task-02-force-logout-on-genuine-failure
- `lib/core/networking/token_cubit.dart` — no changes needed
- `lib/core/networking/services/network_service_impl.dart`, `lib/core/networking/commons/network_service.dart` — the shared singleton stays exactly as-is; this fix routes AROUND it for this one call, it doesn't change it

## Implementation Steps

### Step 1: Build the dedicated client

Create a small, focused class that:
- Constructs its own `Dio` instance scoped to `environment.subdomainUrl(tokenData.subdomain)` (the subdomain is already available on the `TokenData` read at the top of `refreshToken()` — no need to depend on whatever base URL the shared `NetworkService` singleton happens to have configured).
- Applies the same baseline `BaseOptions` sanity (connect/receive timeouts, `validateStatus: (s) => s != null && s < 400`) and, in `kDebugMode` only, the same bad-certificate bypass that `RestNetworkServiceImpl` uses — staging servers have self-signed/expired certs. Don't skip this or refresh will fail against staging in debug builds.
- Does NOT register any `onError`/401-retry interceptor. A 401 (or 400) from this endpoint should surface as a plain, mapped `Failure` — never trigger a second refresh attempt.
- Sends the `POST /oauth/token` request with the same query parameters `LoginRepositoryImpl.refreshToken()` already builds (`grant_type=refresh_token`, `refresh_token`, `redirect_uri`, `client_id`, `client_secret`).
- Maps the response the same way the existing code already deserializes it (`GetTokenDeserializer`) on success.
- On failure, maps the error to a `Failure` (reuse `Failure` subtypes from `lib/core/networking/errors/failures.dart` — do not invent new ones). Specifically: **any HTTP 400 or 401 response from this endpoint must map to `UnauthorizedFailure`** — that's the signal task-02 depends on to detect "the server genuinely rejected the refresh" (as distinct from a network/timeout/5xx failure, which should map to something else, e.g. `MessageFailure`/`UnexpectedFailure` as appropriate). This is a deliberate, narrower rule than the existing generic `ErrorModel.toFailure()` (which only maps HTTP 403 → `UnauthorizedFailure` and doesn't fit OAuth's typical `{"error": "invalid_grant", ...}` body shape) — do not reuse `ErrorModel.toFailure()` unmodified for this endpoint; write the narrower mapping directly in the new client.
- Consider whether any part of `RestNetworkServiceImpl._handleFailure` (the 422/500/generic error-body parsing) is worth extracting into a small shared helper vs. duplicating a minimal subset here. Prefer extraction if it's a clean, low-risk change; otherwise duplicate only what's needed and leave a short comment explaining why this client intentionally doesn't share the interceptor chain (link back to this task/plan).

### Step 2: Wire it into `refreshToken()`

Replace the `networkService.sendRequestCall(...)` call inside `LoginRepositoryImpl.refreshToken()` with a call to the new dedicated client. Keep everything else in that method identical: the early guard on missing/empty `tokenData.refreshToken`, updating `TokenCubit` and `storageManager` on success, and the `Either<Failure, String>` return shape.

### Step 3: Verify no regression to the shared interceptor's own behavior

`RestNetworkServiceImpl._getOrCreateRefreshToken`'s `Completer`-coalescing for concurrent 401s from OTHER requests (tickets, assets, chat, etc.) must keep working exactly as before — this task changes WHERE the refresh HTTP call goes, not the coalescing logic around it. Confirm the existing `refreshToken` callback shape (`Future<String?> Function()`) passed into `RestNetworkServiceImpl` is unchanged; only what happens *inside* `LoginRepositoryImpl.refreshToken()` changes.

## Testing

- [x] New test: simulate the refresh-grant HTTP call itself returning 401 (the scenario that used to deadlock) and assert the call to `refreshToken()` completes within a bounded time (e.g. wrap the awaited call in `.timeout(Duration(seconds: 2))` in the test, or use `fakeAsync`/`FakeAsync` if the existing test suite already uses it) and returns `Left(UnauthorizedFailure(...))` — not a hang.
- [x] New test: successful refresh still updates `TokenCubit` and calls `storageManager.saveToken(...)` with the new token data (same assertions the existing `login_repository_impl_test.dart` already makes for the old code path — port them over).
- [x] New test: a transient failure (e.g. connection error / 500) from the refresh-grant call maps to something other than `UnauthorizedFailure` (so task-02 doesn't force-logout on transient errors).
- [x] Existing `rest_network_service_5xx_test.dart` and any other `RestNetworkServiceImpl`/`_getOrCreateRefreshToken` tests for concurrent-401 coalescing still pass unmodified — this task must not change that behavior.
- [x] Existing tests still pass
- [x] Linter / static analysis passes: `fvm flutter analyze`
- [x] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] Do NOT update `docs/kb-projects/syncro-flutter/technical/integrations/oauth.md` or `ai-patterns/known-issues.md` as part of this task — that's an explicit follow-up outside this plan's scope (see overview.md Scope section). Do not let `check-kb-index`/`document-solution` auto-triggers expand this task's scope; if they fire, note the KB gap in `status.md` instead of editing the stale docs here.
- [ ] If you extracted a new shared helper (e.g. from `_handleFailure`) that's reusable/non-obvious, a short note in `status.md`'s Adaptations section is enough — no new KB doc required for this narrowly-scoped fix.

## Completion Criteria

- [x] Refresh-grant network call for `/oauth/token` no longer shares `RestNetworkServiceImpl`'s Dio instance or 401-retry interceptor
- [x] A refresh-grant call that itself 401s/400s cannot deadlock `_refreshTokenCompleter` (proven by test, not just code review)
- [x] Genuine rejection maps to `UnauthorizedFailure`; transient failure does not
- [x] All tests pass (one pre-existing, unrelated failure in `test/features/chats/chat_models_test.dart` confirmed present on `develop` before this task's changes — not caused by this task)
- [x] No regressions in existing functionality (concurrent-401 coalescing for other feature requests unaffected — `rest_network_service_5xx_test.dart` unmodified and still passes)
- [x] Documentation / KB updates completed or explicitly marked not needed (see above — explicitly not needed for this task)
- [x] Changes committed to `plan/refresh-token-zombie-session/task-01-isolate-refresh-network-call` branch
- [x] Status updated in `status.md`
