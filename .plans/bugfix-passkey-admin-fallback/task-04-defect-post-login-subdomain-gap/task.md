# Task: Defect — passkey login never learns the real subdomain, breaking the post-login `/me` call

**Plan**: bugfix-passkey-admin-fallback
**Task ID**: task-04
**Task Path**: task-04-defect-post-login-subdomain-gap
**Depends On**: "None" (backend prerequisite already shipped — see Root Cause)
**Blocks**: "None"
**JIRA**: N/A
**Severity**: P2
**Found By**: Alex Salazar
**Found During**: Manual staging verification on `ss1`, real device (2026-09-14), after backend MR 19414 (SE-13403) deployed commit `ac8ce8` (global credential-id lookup) — the passkey sign-in itself started succeeding, surfacing this as the next symptom in the same flow.

## Bug Description

After a successful passkey login via the admin-host fallback (this plan's task-01), the app immediately fails with **"Signed in, but we couldn't load your account. Please try again."** (`AppStrings.passkeySignInMeFailedMessage`, `app_strings.dart:307-308`).

The passkey sign-in itself now works (backend resolves the account globally by credential id — MR 19414). But the app never learns the tenant's **real subdomain** from that login, so the very next request — the post-login `/me` fetch — still goes out via the tenant-agnostic admin host, which cannot resolve the account for a subdomain-scoped endpoint like `/api/v1/me`.

## Root Cause

**Files**:
- `lib/features/authentication/passkey/application/passkey_login_cubit.dart:128-136` (`_completeSignIn()`)
- `lib/features/authentication/login/infrastructure/login_repository_impl.dart:178-192` (`me()`)
- `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart:105-144` (`verifyLogin()`, `_ensureNetworkServiceInitialized()`)
- `lib/features/authentication/login/infrastructure/get_token_deserializer.dart`, `lib/features/authentication/login/domain/get_token_response.dart`

**What it does today**:
1. `PasskeyRepositoryImpl.verifyLogin()` calls `_ensureNetworkServiceInitialized()` before the request, which points `networkService` at the **admin host** when no subdomain is persisted (correct, this plan's task-01 behavior).
2. On success, it builds `TokenData(subdomain: AppSharedPreferences.getSubdomain(), ...)` — still empty, since none was ever known. The backend response itself carries no subdomain either, at the time this plan was first built.
3. `_completeSignIn()` then calls `MeUseCase.call(MeParams(isCheckingToken: false))`. Because `isCheckingToken` is `false`, `LoginRepositoryImpl.me()` **skips** its `_initNetworkService(data.subdomain)` branch entirely and just fires the request through whatever `networkService` is currently pointed at — still the admin host.
4. `/api/v1/me` needs a real tenant subdomain in the Host header to resolve the account (confirmed: the route is reachable at `admin.ss1.syncrostaging.com` — it 401s correctly on a bad token — but a valid passkey-issued token still can't resolve an account through it). The call fails, `_completeSignIn()` emits `PasskeyLoginFailed(passkeySignInMeFailedMessage)`.

**Backend fix already shipped** (no longer blocking, do not re-scope): GitLab MR 19414 (SE-13403), commit `07477a6da94fbad274c9a4764070d5933f9ef46f`, deployed and confirmed live on `ss1`. `POST /api/v1/mobile/passkey_sessions` (login-verify) now returns `subdomain: credential.account.subdomain` merged into the token response body, specifically so a client with no prior subdomain context can learn it. This task is entirely client-side.

**What it should do**: Parse the backend's new `subdomain` field, persist it, and re-point `networkService` to the real tenant host *before* `_completeSignIn()`'s `/me` call fires.

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/authentication/login/domain/get_token_response.dart` | modify | Add a `subdomain` field to `GetTokenResponse` |
| `lib/features/authentication/login/infrastructure/get_token_deserializer.dart` | modify | Parse `data['subdomain']` |
| `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` | modify | `verifyLogin()`: persist the real subdomain, re-point `networkService` before returning `Right(data)` |

### Do NOT Modify

- `lib/features/authentication/passkey/application/passkey_login_cubit.dart` — no change expected; once `verifyLogin()` returns having already re-pointed `networkService` correctly, `_completeSignIn()`'s existing `/me` call works unmodified
- `lib/features/authentication/login/application/login_cubit.dart` — the normal WebView code-exchange flow already gets its subdomain from the OAuth redirect URI (`login_cubit.dart:70`), not from the token response body; adding the field to the shared `GetTokenResponse`/`GetTokenDeserializer` must not change that flow's behavior

## Implementation Steps

### Step 1: Thread the `subdomain` field through the shared token response

`GetTokenResponse`/`GetTokenDeserializer` are shared between `LoginRepositoryImpl.getToken()` (normal flow) and `PasskeyRepositoryImpl.verifyLogin()` (this flow). Add `final String? subdomain;` to `GetTokenResponse` and parse `data['subdomain']` in the deserializer. Nullable/optional — the normal flow's backend response also already includes this field (per MR 19414's comment: mirrors `CodeResponseOverrides` in `config/initializers/doorkeeper.rb`), but that flow doesn't need to read it, so this must not require the field to be present in every response.

### Step 2: Persist the real subdomain and re-point `networkService` in `verifyLogin()`

In `PasskeyRepositoryImpl.verifyLogin()`'s success branch (currently around lines 126-139):
- Use `data.subdomain` instead of `AppSharedPreferences.getSubdomain()` when building `TokenData`.
- Call `AppSharedPreferences.setSubdomain(data.subdomain)`, mirroring what `LoginRepositoryImpl._initNetworkService()` does (`login_repository_impl.dart:123`).
- Re-point `networkService` to `environment.subdomainUrl(data.subdomain)` **before** returning `Right(data)`, so it's live before `PasskeyLoginCubit._completeSignIn()` fires its `/me` call.
- Guard against a missing/blank `data.subdomain` (e.g. an older backend still on the pre-MR-19414 response shape) — fall back to today's behavior rather than crashing or re-pointing to an invalid host.

### Step 3: Resolve the refresh-token-callback design question before writing the re-init call

`LoginRepositoryImpl._initNetworkService()` wires a working refresh callback (`() async => (await refreshToken()).fold((_) => null, (token) => token)`) when it re-points `networkService`. `PasskeyRepositoryImpl._ensureNetworkServiceInitialized()` currently uses `() async => null` — correct for the unauthenticated challenge/verify calls, but if the Step 2 re-init reuses it, the immediate post-login `/me` call still succeeds, but any token refresh needed later **in the same app session** (before the next cold start) would silently fail, since `PasskeyRepositoryImpl` has no access to `LoginRepositoryImpl.refreshToken()` today.

Decide and document the approach (e.g. inject `LoginRepository`'s refresh capability into `PasskeyRepositoryImpl`, or another seam) as part of this task — do not silently ship `() async => null` for the post-login re-init without a documented reason if a working refresh path is feasible.

### Step 4: Regression Tests

- `passkey_repository_impl_test.dart`: on a successful `verifyLogin` with a backend response carrying `subdomain`, assert `TokenData.subdomain` matches it (not the empty `AppSharedPreferences.getSubdomain()`), `AppSharedPreferences.setSubdomain` was called with it, and `networkService.initService` was called with the real subdomain's base URL (not the admin host) before the method returns.
- Add/adjust a test for the missing-`subdomain` fallback path (Step 2's guard).
- `get_token_deserializer_test.dart` (or equivalent): parses `subdomain` when present, tolerates its absence.
- Confirm existing `LoginRepositoryImpl.getToken()` tests still pass unchanged with the added field on `GetTokenResponse`.

## Testing

- [ ] Regression test added (fails before fix, passes after)
- [ ] Existing tests still pass
- [ ] `pre-commit-check` passes
- [ ] Original repro steps no longer reproduce the bug — requires real-device retest on `ss1` (backend side already confirmed live)

## Completion Criteria

- [ ] Bug fixed
- [ ] Refresh-token-callback design question (Step 3) resolved and documented, not improvised
- [ ] Regression test added
- [ ] Existing tests pass
- [ ] Changes committed to `plan/bugfix-passkey-admin-fallback/task-04-defect-post-login-subdomain-gap` branch
- [ ] Status updated in `status.md`
- [ ] Real-device retest on `ss1` confirms `/me` succeeds immediately after a fresh-install passkey login
