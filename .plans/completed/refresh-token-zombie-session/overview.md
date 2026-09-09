# Plan: Fix Refresh-Token Zombie Session

**Status**: complete
**Created**: 2026-09-09
**Last Updated**: 2026-09-09
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: plan/refresh-token-zombie-session — populated by `execute-plan` at start; "N/A" when `PR_INTEGRATION=false`
**Base Branch**: develop — developer-confirmed when running `execute-plan`; may be overridden at runtime (corrected 2026-09-09: `main` is vestigial for this project, `develop` is the real integration branch — see `docs/kb-projects/syncro-flutter/AGENTS.md`)

## Objective

Fix a bug where, after the app sits idle long enough for the access token to expire, the session becomes a "zombie": the user still appears logged in but every screen shows empty/no-data widgets, and only a full app restart recovers. Also close the related gap where a genuinely-dead refresh token never forces the user back to login.

## Scope

### In Scope

- Fix the circular-await deadlock in `RestNetworkServiceImpl` that permanently wedges the 401-retry `Completer` once triggered, hanging every subsequent API call for the rest of the app session.
- Force an automatic logout when a token refresh is genuinely rejected by the server (not the deadlock, not "no token present" — a real `invalid_grant`-class rejection).

### Out of Scope

- Updating the stale project KB docs (`docs/kb-projects/syncro-flutter/technical/integrations/oauth.md`, `ai-patterns/known-issues.md`) that currently claim refresh is "not implemented" — flagged as a follow-up, not part of this fix. Run `document-solution`/`check-kb-index` separately once this plan is complete.
- Any UX polish beyond "redirect to login" (e.g. a "session expired" toast/banner) — user explicitly chose silent forced logout over a banner+retry UI for this plan.
- Refactoring `RestNetworkServiceImpl`/`NetworkService` beyond what's needed for the fix (e.g. no general DI/architecture cleanup).

## Root Cause (confirmed by code reading, not yet reproduced live — see engram `bugs/refresh-token-zombie-session`)

1. `RestNetworkServiceImpl` (`lib/core/networking/services/rest_api/rest_network_service.dart`) is a single long-lived instance for the whole app session — only recreated on login or on `me(isCheckingToken:true)` at cold boot (`LoginRepositoryImpl._initNetworkService`).
2. The OAuth refresh call (`POST /oauth/token`, `grant_type=refresh_token`, in `LoginRepositoryImpl.refreshToken()`) is issued through the exact same `NetworkService` singleton / Dio instance / interceptor chain as every other API call — `AppRequests.getToken` resolves to the relative path `/oauth/token`, resolved against the same Dio `baseUrl`.
3. Consequence: `_setHeaders()` unconditionally attaches the current (possibly expired) `Authorization: Bearer` header to the refresh request itself. If that refresh request also comes back 401 (or 400, per OAuth's typical `invalid_grant` shape), `_handleError` fires again **for the refresh request itself** and calls `_getOrCreateRefreshToken()` a second time. Since `_refreshTokenCompleter` is still non-null (the original attempt hasn't resolved), it awaits the *same* completer's future — which can never complete, because the code that would complete it is the very call blocked awaiting it. Permanent deadlock.
4. `_refreshTokenCompleter` stays stuck non-null for the rest of the app's life (the `finally` that clears it never runs, since the awaiting future never resolves or throws). Every subsequent 401 on any feature (tickets, assets, chat) hangs forever awaiting the same broken completer — matching the reported symptom: `AuthenticationCubit`'s own state is never invalidated (nothing calls `logOut()`), so the user still looks "logged in" while every list screen is stuck in its initial/empty state forever, with no error surfaced.
5. Restarting the app creates a brand-new `RestNetworkServiceImpl` (fresh `_refreshTokenCompleter = null`) and reloads the token from secure storage into `TokenCubit` via `GetTokenFromDBUseCase`, breaking the deadlock — the underlying refresh token was never actually invalid server-side, so a normal refresh now succeeds. This is why restart "fixes" it without a real re-login.

## Kill Criteria

- If a separate Dio client for the OAuth token endpoint is found to be incompatible with something the shared interceptor chain currently provides that this endpoint actually needs (unlikely — investigate before abandoning; the compact Dio KB doc explicitly lists "Auth header interceptor" as a per-endpoint concern, not a hard requirement).
- If reproducing the deadlock in a test proves the root cause hypothesis wrong (e.g. the refresh-grant call never actually receives a 401 in practice) — stop and re-investigate before proceeding with Task 1's fix.

## Phases

Flat plan (2 tasks, no phases).

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-isolate-refresh-network-call | Isolate the OAuth refresh call from the shared 401-retry interceptor | adapted | — |
| task-02-force-logout-on-genuine-failure | Force logout when a refresh is genuinely rejected by the server | complete | task-01-isolate-refresh-network-call |

## Branch Convention

Pattern: `plan/refresh-token-zombie-session/{task-path}`

- `plan/refresh-token-zombie-session/task-01-isolate-refresh-network-call`
- `plan/refresh-token-zombie-session/task-02-force-logout-on-genuine-failure`

Base branch: confirmed by developer when running `execute-plan` (see `**Base Branch**` field above)

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `lib/core/networking/services/rest_api/rest_network_service.dart` | Owns the 401-retry interceptor and the `_refreshTokenCompleter` that deadlocks |
| `lib/features/authentication/login/infrastructure/login_repository_impl.dart` | `refreshToken()` — the OAuth refresh-grant call; both fixes land here |
| `lib/features/authentication/application/authentication_cubit.dart` | Cold-boot auth check; calls `loginRepository.refreshToken()` and already owns `logOut()` |
| `lib/core/networking/token_cubit.dart` | In-memory token state, read fresh by both the interceptor and the repository |
| `lib/core/networking/errors/failures.dart`, `error_model.dart` | Existing `UnauthorizedFailure` type reused (not invented) to signal genuine rejection |

## Risks

- The refresh-grant response body shape for a genuine OAuth rejection (`invalid_grant`) is assumed based on typical OAuth2/Doorkeeper conventions but hasn't been confirmed against a real server response in this session — mitigate by having task-01 log/inspect the actual failure shape during manual verification, not just guess from the generic error path.
- Forcing `AuthenticationCubit.logOut()` from inside `LoginRepositoryImpl` (repository layer reaching into the application-layer cubit via GetIt) is a layering shortcut — but it's consistent with the existing pattern already in this file (`TokenCubit` is read/written the same way at lines 117/146), so it's not a new violation, just an extension of the established convention.

## Defects

<!-- Bugs discovered during plan execution. Added by execute-task's Bug Discovery Protocol (Step 5a). -->

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [x] A refresh-grant request that itself returns 401/400 can no longer deadlock `_refreshTokenCompleter` — proven by a test with a bounded timeout, not just manual observation. Verified independently (not just agent-reported): `auth_token_client_test.dart` + `login_repository_impl_test.dart`, the 401-deadlock-proof test wrapped in `.timeout()`, asserts `Left(UnauthorizedFailure)`.
- [x] All other API traffic continues to share the single `Completer`-coalesced refresh behavior for concurrent 401s (existing behavior preserved, not regressed). `rest_network_service.dart` was untouched by this plan (confirmed via `git diff --stat` on both task branches) — no automated test exists for this specific behavior even pre-plan (`rest_network_service_5xx_test.dart` is a documentation-only stub, no executable tests, per its own header comment — unrelated pre-existing gap, not introduced here). Verified by code review / zero diff to the file, not by a new test.
- [x] A genuinely-rejected refresh (`UnauthorizedFailure`) triggers `AuthenticationCubit.logOut()` automatically, both from the reactive (mid-session) path and the cold-boot path. Verified via `login_repository_impl_test.dart`'s `verify(mockAuthenticationCubit.logOut())` assertions; both call paths funnel through the same `LoginRepositoryImpl.refreshToken()` method that was changed.
- [x] A refresh that fails for a transient reason (network error, 5xx, no connectivity) does NOT force a logout — only genuine rejection does. Verified via `verifyNever(...)` assertions for the transient-500 and no-token-guard cases.
- [x] All existing tests pass; new tests cover both fixes. Independently verified: `fvm flutter analyze` clean, targeted suites pass, full suite 2300/2301 — the 1 failure (`chat_models_test.dart`, a pre-existing string-casing mismatch) is unrelated and pre-dates this plan (confirmed via `git stash` by task-01's agent).
- [x] KB/documentation updates complete or explicitly marked not needed. Done in this session, after plan completion: `docs/kb-projects/syncro-flutter/technical/integrations/oauth.md` and `ai-patterns/known-issues.md` corrected to reflect the real implementation and this fix; `README.md` index dates updated.

## References

- **JIRA Epic**: [SE-13836](https://syncrotech.atlassian.net/browse/SE-13836)
- **Related Plans**: None
- **Engram discovery**: `bugs/refresh-token-zombie-session` (project: syncro-flutter) — full investigation notes and evidence trail
