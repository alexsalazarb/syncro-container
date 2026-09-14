# Plan: Passkey login fails on fresh install with no persisted subdomain

**Status**: complete — all 4 tasks + the direct dashboard fix done, staging verification passed end-to-end on both iOS and Android (2026-09-14, real device, `ss1`, exact original repro). Squash-merged into `develop` (commit `2c24b1f9`), version bumped to `1.8.0+450`, `develop`/`qa` pushed to both `origin` and `bla`, QA build triggered on Bitrise.

**2026-09-14 (later same day)**: real-device retest on `ss1` with task-04's fix got past the `/me` failure (passkey login itself now works end-to-end) but surfaced a second, unrelated bug on the same screen: `DashboardPage` offered "Set Up Passkey" (`PasskeyEnrollmentOffer`) immediately after a successful passkey sign-in. Root cause: the enrollment-offer call was missing the same `!loggedInViaPasskey` guard already applied to the MFA warning two lines above it in `dashboard_page.dart:initState` — `AppSharedPreferences.getPasskeyEnrolledOnThisDevice()` only tracks whether *this app install* ran the enrollment ceremony, so a fresh install signing in via an OS-level-synced passkey never sets it, leaving the offer's only real eligibility check to pass. Fixed directly (Alex's call — small, obvious fix, skipped the formal defect-task ceremony): added the guard, rewrote the existing test that had locked in the buggy behavior as "expected," added a second test for the inverse case. Same commit range as task-04 (`plan/bugfix-passkey-admin-fallback/task-04-defect-post-login-subdomain-gap` branch, still local/unpushed). `flutter analyze` clean, dashboard tests 6/6 green.

**2026-09-11**: added task-03 (defect) — verifying the `credential_not_recognized` snackbar copy surfaced a related bug: the cubit clears local enrollment flags on that error code unconditionally, which is only a safe signal when the request went through a known tenant subdomain. Via the admin-host fallback (this plan's own change), the same code can fire from the still-open backend gap, not a real revocation. Independent of Justin's fix — worth doing either way.

**2026-09-11**: squash-merged tasks 1-2 into `develop` locally as a single commit (`fix(passkey): fall back to shared admin host when no subdomain is persisted`, no Jira ticket — conventional-commit style, matching this repo's convention for non-ticketed fixes) to get a QA-buildable candidate ready. `flutter analyze` clean, 109/109 tests green on `develop`. This does NOT mean the plan is done — staging verification is still blocked on the backend fix above; merging now is a deliberate "get ahead of it" call so the client fix ships automatically the moment the backend issue is resolved, without waiting on another mobile release cycle.

**2026-09-11 (later same day)**: task-03 folded into that same `develop` commit via `git commit --amend` (not a second commit — the earlier one wasn't pushed yet, so amending doesn't rewrite shared history), per Alex's request to keep `develop`'s history to one commit for this whole plan. All 3 tasks' code now live in a single local `develop` commit. Still not pushed to remote. `flutter analyze` clean, 110/110 tests green.
**Created**: 2026-09-10
**Last Updated**: 2026-09-14
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
- ~~Justin (backend) confirms the RP ID is actually scoped per-tenant/subdomain, or that no login-challenge/options endpoint is reachable at the admin host~~ — **RESOLVED 2026-09-10**: Justin confirmed the admin host is the shared RP ID ("Yes 👍 It should be. If it's not working, let me know."). This criterion no longer applies.

## Task Summary

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-admin-host-fallback | Fall back to admin host when no subdomain is persisted | complete | — |
| task-02-regression-tests | Rewrite/add passkey login tests for the fallback path | complete | task-01-admin-host-fallback |
| task-03-defect-false-clear-on-admin-fallback | Defect: don't clear local enrollment on `credential_not_recognized` via the admin-host fallback | complete | — |
| task-04-defect-post-login-subdomain-gap | Defect: passkey login never learns the real subdomain, breaking the post-login `/me` call | complete | — |

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
| task-03-defect-false-clear-on-admin-fallback | `credential_not_recognized` via admin-host fallback incorrectly clears local enrollment flags, same ambiguity as the still-open backend gap | Manual staging verification (real-device test, 2026-09-10) | — | complete |
| task-04-defect-post-login-subdomain-gap | Passkey login never learns the real subdomain post-login; `/me` still hits the admin host and fails with "couldn't load your account" | Manual staging verification (real-device test, ss1, 2026-09-14), after backend MR 19414 commit `ac8ce8` fixed sign-in itself | — | complete |

## Completion Checklist

<!-- Verified by execute-task when the last task completes. Do not remove items. -->
- [x] All tasks complete or adapted
- [x] Bug no longer reproducible with original repro steps — **VERIFIED 2026-09-14**: real device, ss1, exact original repro (delete app, reinstall, select ss1, sign in with a passkey enrolled some time ago) — sign-in succeeded, Dashboard loaded with no "couldn't load your account" error, no false "Set Up Passkey" offer. Confirmed on both iOS and Android.
- [x] Regression test: red before fix, green after (verified)
- [x] All existing tests pass
- [x] investigation.md root cause matches the actual fix (no drift)
- [x] All consumers handle the changed behavior (if applicable) — only consumer is `PasskeyLoginButton`, unaffected (calls `signIn()` the same way regardless)
- [x] KB/documentation updated or explicitly marked not needed
- [x] Ticket transitioned (or transition noted for manual action) — N/A, no ticket
- [x] Staging verification complete — **DONE 2026-09-14**, both platforms, on `ss1`. Full timeline:
  1. iOS Simulator attempt (ss1) failed with a webcredentials association error — false negative, Simulator-only limitation (see [[passkey-simulator-associated-domains-unreliable]]); client entitlements + backend AASA both independently verified correct.
  2. Real physical device (iOS), controlled sequence: enrolled a passkey on `ss1`, fully deleted and reinstalled the app, tapped "Sign in with Passkey" — native sign challenge succeeded, but `PasskeyRepository.verifyLogin` returned `credential_not_recognized` against the admin host. Reproducible, controlled finding — reported to Justin for backend investigation.
  3. Backend fix (MR 19414, SE-13403, commit `ac8ce8`): global credential-id lookup, no longer tenant-scoped. Confirmed via unit/request specs reproducing this exact scenario.
  4. Real-device retest: sign-in itself succeeded, but surfaced task-04's gap (`/me` still hit the admin host, "couldn't load your account") — backend shipped `subdomain` in the response (commit `07477a6d`), client consumed it (task-04).
  5. Real-device retest again: surfaced the enrollment-offer bug (fixed directly, same session, see 2026-09-14 changelog entry above).
  6. **Final verification, real device, exact original repro** (delete app, reinstall, select ss1, sign in with a passkey enrolled some time ago): succeeded end-to-end on both iOS and Android — sign-in works, Dashboard loads cleanly, no false enrollment offer.

## Revert Plan

**Revert trigger**: Staging verification shows the admin host does not accept passkey login challenges for arbitrary accounts (i.e., Kill Criteria assumption was wrong), or the fallback produces a worse UX than the current explicit failure message.
**Revert steps**: Revert task-01's commit(s) on `main`; the old short-circuit behavior (and its original test) is fully restored by a straight `git revert`.
**Rollback owner**: Alex Salazar

## References

- **Ticket**: N/A
- **Investigation**: [investigation.md](investigation.md)
- **Related Plans**: `.plans/completed/SE-13402-passkey-login/` (dangling, not in `main` history — see investigation.md Step 2b)
