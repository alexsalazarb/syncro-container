# Investigation: Passkey login fails with no subdomain persisted

**Plan**: bugfix-passkey-admin-fallback
**Date**: 2026-09-10
**Trigger**: Internal discovery — follow-up conversation with Justin (backend) about whether passkey login could work as a true "discoverable, no password fallback" experience on a fresh install / new device.

## KB Check (3-layer)

- **Layer 1/2/3**: No passkey/WebAuthn KB doc exists anywhere under `docs/`. Code comments (`native_passkey_provider.dart`, `mock_passkey_repository_impl.dart`) reference a `docs/kb-projects/syncro-flutter/technical/integrations/passkey-webauthn-contract.md` that was supposed to be imported by SE-13402 task-02 — it does not exist in the current tree (see Feature History Trace below). **This is a KB gap** (per `KB_GAP_PROTOCOL.md`): the passkey feature shipped to `main` with zero KB documentation. Flagged to the user; recommend `document-solution` after this fix lands to close the gap, covering both the RP ID/admin-host model and this fix.
- Proceeded with direct code exploration since no KB doc covers this area.

## Code Exploration

The `subdomain` gate is a client-side implementation shortcut, not a WebAuthn/RP-ID requirement.

1. **`subdomain` is never used to build the RP ID.** It is used exclusively to construct the backend REST API host:
   - `environment.dart:120` — `String subdomainUrl(String subdomain) => 'https://$subdomain.$basePath';`
   - `passkey_repository_impl.dart:178-185` — `_ensureNetworkServiceInitialized()` reads `AppSharedPreferences.getSubdomain()` and calls `networkService.initService(environment.subdomainUrl(subdomain), ...)`. **If `subdomain` is empty, this method silently returns without initializing anything** (line 180: `if (subdomain.isEmpty) return;`).

2. **The `rpId` comes entirely from the backend's challenge response** and passes through unchanged:
   - `passkey_login_cubit.dart:92-93` — `nativePasskeyProvider.signChallenge(rpId: challenge.rpId, ...)`.
   - `native_passkey_provider.dart:319` — `AuthenticateRequestType(relyingPartyId: rpId, ...)`.
   - The client never derives `rpId` from the local subdomain.

3. **There is already a tenant-agnostic "admin" host used today, with zero subdomain known:**
   - `environment.dart:118` — `String get adminUrl => "https://admin.$basePath/oauth/authorize";`
   - `login_repository_impl.dart:65` — the standard "Sign in" (OAuth WebView) button hits this exact host before any subdomain is known; the subdomain is only learned afterward from the redirect callback (`login_cubit.dart:69-70`, `login_repository_impl.dart:123`).
   - Note: `environment.subdomainUrl('admin')` produces the exact same host (`https://admin.$basePath`) as `adminUrl`'s host portion — this is the mechanism the fix reuses (see task-01).

4. **The passkey contract/mock already models the RP ID with that same `admin.$basePath` pattern:**
   - `mock_passkey_repository_impl.dart:53,69` — both enrollment and login challenge fixtures use `rpId: 'admin.ss1.syncrostaging.com'`.

5. **The exact gate:**
   - `passkey_login_cubit.dart:63-69`, inside `PasskeyLoginCubit.signIn()`:
     ```dart
     if (AppSharedPreferences.getSubdomain().isEmpty) {
       logger('Passkey login skipped: no subdomain persisted on this environment');
       safeEmit(
         const PasskeyLoginFailed(AppStrings.passkeySignInNoCredentialMessage),
       );
       return;
     }
     ```
   - The comment directly above (lines 53-62) states the actual intent: this exists only to produce a friendly `PasskeyFailure` message *before* `_ensureNetworkServiceInitialized()`'s silent no-op turns the failure into a generic, misleading `MessageFailure('Network Service not initialized')`. **It is a UX patch for a missing fallback, not a security boundary.**

6. **The passkey button is already unconditionally visible on the login screen**, with no subdomain/tenant gating in the UI:
   - `passkey_login_button.dart` doc comment: visibility is deliberately not restricted, "matching how Google/GitHub surface their own 'Sign in with a passkey' entrypoints."
   - `login_view.dart` / `login_buttons.dart` — no subdomain/company field exists anywhere on this screen.

## Hypothesis Validation

**Confirmed.** The original hypothesis (RP ID might be scoped per-tenant, making this architecturally impossible) was **wrong** — corrected via Justin's pushback and this code investigation. The client already has everything needed to hit a shared admin host; it just gives up instead of trying.

## Step 2a: Origin Analysis

- **Introducing commit**: `892b01bc` (`SE-13402: Mobile App: Passkey/Biometric Login — Native Enrollment & Sign-In`) — a single squashed merge to `main` (2026-09-07) containing the entire passkey feature's commit history.
- **Specific sub-commit** (per squashed message, titled `fix(passkey): show the correct message on a never-logged-in-here environment`): this exact gate was added as a direct, deliberate consequence of an earlier decision in the same squash — `feat(passkey): always show the Sign in with Passkey entrypoint` — which stopped gating the passkey button's visibility on local enrollment state. Once the button became reachable from an environment the user had never logged into with a password, tapping it hit `_ensureNetworkServiceInitialized()`'s silent no-op and surfaced a confusing generic network error. The gate at lines 63-69 fixed *that* symptom by failing fast with a better message — but did not consider falling back to the already-existing tenant-agnostic admin host as an alternative to failing.
- **Origin classification**: **Side effect of a prior change.** The "always show entrypoint" decision was correct (matches the intended UX); the response to the resulting edge case (no persisted subdomain) chose the wrong fallback (fail with a friendly message) instead of the available one (initialize against the admin host, same as the OAuth login flow already does).

## Step 2b: Feature History Trace

- `git log --all` for `.plans/completed/SE-13402-passkey-login/` shows a full 15-task plan (`01a3a90` → `d2f388a`, archived 2026-09-07) documenting the entire passkey build, including the RP ID/admin-domain wiring (`02905b1`: "permanent 13-domain association wiring", covering `admin.syncromsp.com` + `admin.ss1-12.syncrostaging.com` — i.e., explicit confirmation the RP ID's associated domains are already provisioned as this exact shared admin-per-environment set, not per-tenant).
- **Discrepancy found**: commit `d2f388a` is **not an ancestor of current `main`** (`git merge-base --is-ancestor d2f388a HEAD` → not an ancestor). The plan/KB-documentation branch that produced this archive was never merged into `main`, even though the squashed code commit (`892b01bc`) was. This explains why `.plans/completed/SE-13402-passkey-login/` and the WebAuthn contract KB doc referenced in code comments are both absent from the working tree today, despite the code itself (and its comments referencing them) being present. **Out of scope for this bug fix** — flagged to the user as a separate housekeeping gap (recovering or re-documenting the SE-13402 plan/KB artifacts), not blocking this plan.
- **Key takeaway for this fix**: the domain-association work in that dangling history (`02905b1`) independently corroborates that the RP ID's associated domains are the per-environment admin hosts, not per-tenant subdomains — consistent with what the mock/contract already models.

## Step 2c: Test Coverage Analysis

- `passkey_login_cubit_test.dart:376-397` has an **existing, explicit regression test** for the current (buggy) behavior:
  ```dart
  blocTest<PasskeyLoginCubit, PasskeyLoginState>(
    'emits [InProgress, Failed] with the "no passkey" message ... when this
    device has never logged in on the current environment (no subdomain
    persisted, ...)',
    setUp: () { AppSharedPreferences.setSubdomain(''); },
    ...
    expect: () => [PasskeyLoginInProgress(), const PasskeyLoginFailed('No passkey was found...')],
    verify: (_) { verifyNever(mockPasskeyRepository.getLoginChallenge()); },
  );
  ```
  This test currently passed because it was written to lock in the exact behavior this bug plan is changing. It is not a coverage gap — it is a test that must be **deliberately rewritten** as part of task-02, since its assertion (`verifyNever(getLoginChallenge())`) directly contradicts the new intended behavior (attempt the login challenge against the admin host instead of short-circuiting).
- No test currently exercises `PasskeyRepositoryImpl._ensureNetworkServiceInitialized()`'s empty-subdomain branch in isolation with an assertion on *which* host gets initialized — task-02 must add one.

## Step 2d: Cross-Project Assessment

**Single-project.** All code changes are confined to `syncro-flutter` (`passkey_login_cubit.dart`, `passkey_repository_impl.dart`, and their tests). No backend repository exists in this container; Justin's confirmation is external coordination, not a code dependency tracked as a plan task. `Master Plan`: None.

## Step 2e: Confidence Assessment

**Root Cause Confidence: High (>85%).** The client-side mechanism is fully verified in code with file:line citations — the subdomain gate demonstrably has nothing to do with the RP ID, and a working tenant-agnostic admin host already exists and is exercised today by the OAuth login flow.

**Caveat (explicitly acknowledged, not raising this below High per user instruction):** the fix's correctness still depends on Justin (backend) confirming that the *real* (non-mocked) WebAuthn RP ID is genuinely the shared `admin.$basePath` host for every account in a given environment — not per-tenant — and that a login-challenge/options endpoint is reachable directly at that host. Per the archived (but unmerged) SE-13402 domain-association work (`02905b1`), this is very likely true, and the user has directed this plan to proceed treating it as confirmed. If Justin's actual answer contradicts this, see Kill Criteria in `overview.md`.
