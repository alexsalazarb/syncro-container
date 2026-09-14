# Task: Fall back to admin host when no subdomain is persisted

**Plan**: bugfix-passkey-admin-fallback
**Task ID**: task-01
**Task Path**: task-01-admin-host-fallback
**Depends On**: None
**Ticket**: N/A

## Objective

Make `PasskeyLoginCubit.signIn()` attempt passkey login against the shared admin host (`environment.subdomainUrl('admin')`) when no subdomain is persisted, instead of short-circuiting to `PasskeyLoginFailed`.

## Context

Read [investigation.md](../investigation.md) for full root cause context before starting. Summary:

- `passkey_login_cubit.dart:63-69` currently short-circuits when `AppSharedPreferences.getSubdomain()` is empty.
- `passkey_repository_impl.dart:178-185`'s `_ensureNetworkServiceInitialized()` silently no-ops on an empty subdomain (`if (subdomain.isEmpty) return;`) — this is what the cubit's gate was working around.
- `environment.subdomainUrl('admin')` already produces exactly the host the OAuth login flow uses (`environment.adminUrl`'s host portion, `https://admin.$basePath`) and matches the RP ID pattern the passkey mock/contract already models (`admin.$basePath`).
- **Do not build a new host string or touch `environment.dart`** — `subdomainUrl('admin')` is already the right primitive.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase origin develop`
- [ ] Confirm with the user that Justin has confirmed (or the user has explicitly instructed to proceed on the assumption) that the backend's WebAuthn RP ID is the shared `admin.$basePath` host per environment, per this plan's Kill Criteria — do not implement against an unconfirmed assumption without that explicit go-ahead
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read [investigation.md](../investigation.md) for full root cause context
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/authentication/passkey/application/passkey_login_cubit.dart` | modify | Remove/replace the empty-subdomain short-circuit (lines 63-69) |
| `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` | modify | `_ensureNetworkServiceInitialized()` falls back to `'admin'` instead of no-op'ing on empty subdomain |

### Do NOT Modify

- `lib/core/configs/environment.dart` — no change needed; `subdomainUrl()` already covers this
- `test/features/authentication/passkey/**` — owned by task-02-regression-tests

## Implementation Steps

### Step 1: Fall back to the admin host in `PasskeyRepositoryImpl`

In `passkey_repository_impl.dart`, change `_ensureNetworkServiceInitialized()` so that when `AppSharedPreferences.getSubdomain()` is empty, it initializes the network service against the admin host instead of returning early:

```dart
void _ensureNetworkServiceInitialized() {
  final subdomain = AppSharedPreferences.getSubdomain();
  final host = subdomain.isEmpty ? 'admin' : subdomain;
  networkService.initService(
    environment.subdomainUrl(host),
    () async => null,
  );
}
```

Update the doc comment above it (currently explains why the subdomain-based bootstrap is needed) to also explain the admin-host fallback and why it's safe (RP ID is shared per-environment, not per-tenant — reference investigation.md).

### Step 2: Remove the short-circuit in `PasskeyLoginCubit.signIn()`

In `passkey_login_cubit.dart`, delete the `if (AppSharedPreferences.getSubdomain().isEmpty) { ... return; }` block (lines 63-69) so `signIn()` always proceeds to `passkeyRepository.getLoginChallenge()`. Update or remove the comment above it (lines 53-62) that explained the old rationale — replace with a short note that the empty-subdomain case now falls through to the admin host (see `PasskeyRepositoryImpl`).

Leave the existing failure-handling path (`challengeResult.fold(...)`, `_messageForFailure`) untouched — if the admin host genuinely has no credential for this user, the native `NativePasskeyNoCredentialException` path (already implemented) still produces the same friendly "No passkey was found on this device" message, just reached through the real flow instead of a short-circuit.

### Step 3: Check for a shared constant opportunity (optional, don't over-engineer)

`'admin'` as a literal subdomain now appears in `environment.dart`'s `adminUrl` construction and in `passkey_repository_impl.dart`. If it's a trivial one-line extraction (e.g., a `static const kAdminSubdomain = 'admin';` in `environment.dart`), do it. If it requires touching call sites beyond these two files, skip it — not in scope for this fix.

## Testing

- [ ] Existing tests still pass (except the one task-02 will rewrite for the old short-circuit behavior — coordinate, don't duplicate work)
- [ ] `pre-commit-check` passes
- [ ] Manually verify (or note for staging verification) that `environment.subdomainUrl('admin')` resolves to the same host format Justin confirmed as the RP ID host

## Documentation / KB Updates

- [ ] No new KB doc required for this task specifically — the broader passkey KB gap (no WebAuthn contract doc exists in `docs/`, see investigation.md) is flagged as separate housekeeping, not blocking
- [ ] If this task reveals a new non-obvious pattern beyond what's already documented in investigation.md, run `document-solution`

## Completion Criteria

- [ ] Passkey login with no persisted subdomain reaches `passkeyRepository.getLoginChallenge()` against the admin host instead of short-circuiting
- [ ] Passkey login with a persisted subdomain is unaffected (still uses the tenant host)
- [ ] Documentation / KB updates completed or explicitly marked not needed
- [ ] Changes committed to `plan/bugfix-passkey-admin-fallback/task-01-admin-host-fallback` branch
- [ ] Status updated in `status.md`
