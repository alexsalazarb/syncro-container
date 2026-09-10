# Plan: Passkey login fails on fresh install with no persisted subdomain

**Status**: complete — manual staging verification pending (also pending Justin's backend RP-ID confirmation, see Kill Criteria)
**Created**: 2026-09-10
**Last Updated**: 2026-09-10
**Type**: Bug Fix (Type 3)
**Severity**: P3
**Ticket**: N/A
**Assigned Dev**: Alex Salazar
**Assigned QA**: unassigned
**Trigger**: Internal discovery — follow-up conversation with Justin (backend) about discoverable passkey login without a subdomain
**Blast Radius**: Unknown (edge case) — users with an existing passkey credential (own prior enrollment, or synced via iCloud Keychain/Google Password Manager) attempting to sign in on a device/environment that has never had a password login here (fresh install, factory reset, new device)
**Master Plan**: None

## Bug Summary

`PasskeyLoginCubit.signIn()` hard-fails with a "no passkey" message whenever no `subdomain` is locally persisted, instead of falling back to the app's existing tenant-agnostic admin host — so passkey login can never work as a true "discoverable, no password fallback" experience on a fresh install or new device, even when a valid credential already exists in the user's password manager.

## Root Cause

`passkey_login_cubit.dart:63-69` short-circuits to `PasskeyLoginFailed` when `AppSharedPreferences.getSubdomain()` is empty. This exists only to avoid a confusing generic error from `PasskeyRepositoryImpl._ensureNetworkServiceInitialized()` (`passkey_repository_impl.dart:178-185`), which silently no-ops when the subdomain is empty. The subdomain is only ever used to pick the REST API host (`environment.subdomainUrl()`) — never the WebAuthn RP ID, which always comes from the backend's challenge response. The app already has a shared, tenant-agnostic admin host (`environment.adminUrl`, `admin.$basePath`) used today by the standard OAuth login flow with zero subdomain known, and the passkey mock/contract already models the RP ID with that same host pattern. Full details in [investigation.md](investigation.md).

## Affected Systems

| System | Role | Impact |
|--------|------|--------|
| `PasskeyLoginCubit` | Orchestrates passkey sign-in | Gains a fallback path instead of an early failure |
| `PasskeyRepositoryImpl` | Initializes the network client per tenant | Gains an admin-host fallback when no subdomain is known |
| `Environment` | Provides host-building helpers | Unchanged — `subdomainUrl('admin')` already produces the needed host |
| Backend WebAuthn RP (Justin's team) | Issues challenges, validates RP ID | External dependency — must confirm RP ID is shared per-environment, not per-tenant (assumed confirmed per user direction) |

## Scope

### In Scope
- Fall back to the admin host (`environment.subdomainUrl('admin')`) when initializing the network service for passkey calls if no subdomain is persisted
- Remove the early-return short-circuit in `PasskeyLoginCubit.signIn()` so the login-challenge request is actually attempted against the admin host
- Rewrite the existing test that locks in the old (buggy) short-circuit behavior, and add coverage for the new fallback path
- Regression test that tenant-scoped passkey login (subdomain already persisted) is unaffected

### Out of Scope
- Passkey **enrollment** flow — unaffected; enrollment always happens after a password login, so a subdomain is always already persisted by then
- Recovering or re-documenting the dangling SE-13402 plan/KB artifacts (`.plans/completed/SE-13402-passkey-login/`, WebAuthn contract KB doc) — flagged as a separate housekeeping gap in investigation.md, not blocking this fix
- Backend changes — this plan assumes Justin confirms the RP ID is already the shared admin host; no backend code lives in this container

## Kill Criteria

- Fix introduces worse behavior than the original bug
- Root cause is disproved by new evidence
- **Justin (backend) confirms the RP ID is actually scoped per-tenant/subdomain, or that no login-challenge/options endpoint is reachable at the admin host** — this is the core assumption the fix depends on; if disconfirmed, this plan must be revised or shelved before task-01 starts

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-admin-host-fallback | Fall back to admin host when no subdomain is persisted | complete | — |
| task-02-regression-tests | Rewrite/add passkey login tests for the fallback path | complete | task-01-admin-host-fallback |

## Branch Convention

Task branches: `plan/bugfix-passkey-admin-fallback/{task-path}`
Merge target: `develop` (syncro-flutter's actual integration branch — `main` is vestigial, see `AGENTS.md` "Things to Avoid" #12)

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `lib/features/authentication/passkey/application/passkey_login_cubit.dart` | **Primary fix** — remove the empty-subdomain short-circuit (lines 63-69) |
| `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` | **Primary fix** — `_ensureNetworkServiceInitialized()` fallback to admin host (lines 178-185) |
| `lib/core/configs/environment.dart` | Reference only — `subdomainUrl()`/`adminUrl` already provide what's needed, no change expected |
| `test/features/authentication/passkey/application/passkey_login_cubit_test.dart` | Existing test (lines 376-397) locks in old behavior; must be rewritten |
| `test/features/authentication/passkey/infrastructure/passkey_repository_impl_test.dart` | Add coverage for the admin-host fallback |

## Success Criteria

- [x] Passkey login on a fresh install / never-logged-in-here environment reaches the admin host and attempts the login challenge, instead of short-circuiting
- [x] Regression test fails before fix, passes after (observed directly: both rewritten tests failed pre-task-01, pass post-task-01)
- [x] All existing tests pass (with the intentionally-changed test explicitly rewritten, not just deleted) — 109/109 green
- [x] Tenant-scoped passkey login (subdomain already persisted) is unaffected
- [x] Required KB / documentation updates complete or explicitly marked not needed — explicitly marked out of scope (see Scope section); the broader passkey KB gap is separate housekeeping

## Defects

<!-- Bugs discovered during plan execution. Added by execute-task's Bug Discovery Protocol (Step 5a). -->

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Completion Checklist

<!-- Verified by execute-task when the last task completes. Do not remove items. -->
- [x] All tasks complete or adapted
- [ ] Bug no longer reproducible with original repro steps — **cannot verify without a real device/backend**; the original repro is a fresh install, which unit tests can't reproduce. Pending staging verification.
- [x] Regression test: red before fix, green after (verified)
- [x] All existing tests pass
- [x] investigation.md root cause matches the actual fix (no drift)
- [x] All consumers handle the changed behavior (if applicable) — only consumer is `PasskeyLoginButton`, unaffected (calls `signIn()` the same way regardless)
- [x] KB/documentation updated or explicitly marked not needed
- [x] Ticket transitioned (or transition noted for manual action) — N/A, no ticket
- [ ] Staging verification complete — **requires Justin's backend confirmation first** (see Kill Criteria); must be tested against a real backend, not the mock

## Revert Plan

**Revert trigger**: Staging verification shows the admin host does not accept passkey login challenges for arbitrary accounts (i.e., Kill Criteria assumption was wrong), or the fallback produces a worse UX than the current explicit failure message.
**Revert steps**: Revert task-01's commit(s) on `main`; the old short-circuit behavior (and its original test) is fully restored by a straight `git revert`.
**Rollback owner**: Alex Salazar

## References

- **Ticket**: N/A
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: `.plans/completed/SE-13402-passkey-login/` (dangling, not in `main` history — see investigation.md Step 2b)
