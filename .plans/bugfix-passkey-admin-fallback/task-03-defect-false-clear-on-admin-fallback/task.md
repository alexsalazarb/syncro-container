# Task: Defect — false-positive enrollment clear on admin-host fallback

**Plan**: bugfix-passkey-admin-fallback
**Task ID**: task-03
**Task Path**: task-03-defect-false-clear-on-admin-fallback
**Depends On**: "None"
**Blocks**: "None"
**JIRA**: N/A
**Severity**: P2
**Found By**: Alex Salazar
**Found During**: Manual staging verification of task-01/task-02, real-device test (2026-09-10) that reproduced `credential_not_recognized` against the admin-host fallback

## Bug Description

`PasskeyLoginCubit._signAndVerify()`'s failure handling treats the backend error code `credential_not_recognized` as a definitive "this credential no longer exists, clear local enrollment state" signal, and calls `AppSharedPreferences.clearPasskeyEnrollment()`. Before task-01, this was always safe: the subdomain was always known (enrollment requires a prior password login), so a `credential_not_recognized` response always came from a request to the correct tenant — a reliable signal of genuine revocation.

Task-01 introduced the admin-host fallback for when no subdomain is persisted. We now have direct real-device evidence (see investigation.md and overview.md "2026-09-10" test log) that hitting the admin host can return `credential_not_recognized` for a credential that is demonstrably still valid (confirmed via a second enroll attempt correctly saying "you already have a passkey") — because the unauthenticated passkey endpoints apparently resolve tenant context by host, and the admin host doesn't correctly find the credential regardless of whether it's real. This is being investigated by Justin (backend) separately, but is not yet fixed.

**Consequence**: any device that reaches this code path with an empty subdomain (i.e., every fresh-install passkey login attempt right now) that gets `credential_not_recognized` will have its local enrollment flags wiped — even if they're already empty (fresh install, harmless today), the same code path fires whenever the admin-host fallback is used, and would incorrectly wipe **known-good** local flags on any device that reaches this state with prior valid local enrollment data (e.g., a future scenario where local flags persist across an environment reset but subdomain doesn't). The client currently cannot distinguish "genuinely revoked" from "backend routing bug on the tenant-agnostic path" — same error code covers both.

## Root Cause

**File**: `lib/features/authentication/passkey/application/passkey_login_cubit.dart`, `_signAndVerify()`'s `verifyResult.fold` failure branch (originally lines ~109-120 per investigation.md, adjacent to the `credential_not_recognized` check)
**What it does**: Clears `AppSharedPreferences.clearPasskeyEnrollment()` unconditionally whenever `failure.code == 'credential_not_recognized'`, regardless of whether the request went through a known tenant subdomain or the admin-host fallback.
**What it should do**: Only trust `credential_not_recognized` as a genuine revocation signal when the request was made with a known subdomain (the historically-safe case). When the admin-host fallback was used (no subdomain persisted at request time), do NOT clear local flags on this code — the signal is not reliable there given the known backend gap.

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/authentication/passkey/application/passkey_login_cubit.dart` | modify | Guard the `clearPasskeyEnrollment()` call on `credential_not_recognized` behind a "subdomain was known" check |

### Do NOT Modify

- `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` — no change needed here; the cubit already has access to `AppSharedPreferences.getSubdomain()` directly

## Implementation Steps

### Step 1: Guard the clear

In `_signAndVerify()`, before calling `AppSharedPreferences.clearPasskeyEnrollment()` on `credential_not_recognized`, check `AppSharedPreferences.getSubdomain().isEmpty`. If empty (meaning this request went through the admin-host fallback), skip the clear and log why. If not empty (known tenant), keep the existing clear behavior unchanged.

Update the surrounding comment to explain the distinction and link back to this defect / the admin-fallback backend gap (investigation.md).

### Step 2: Regression Test

In `passkey_login_cubit_test.dart`, add/adjust:
- A test with `AppSharedPreferences.setSubdomain('')` (admin-fallback path) where `verifyLogin` returns `credential_not_recognized` — assert the failure message is unchanged, but `getPasskeyEnrolledOnThisDevice()`/`getPasskeyCredentialId()` are **not** cleared if they were previously set (use a non-empty prior state to make the assertion meaningful, not just "still empty because it started empty").
- Confirm the existing test at "emits [InProgress, Failed] when the server rejects a revoked credential" (known-subdomain case) still passes unchanged — this is the regression guard that the fix must not weaken.

## Testing

- [ ] Regression test added (fails before fix, passes after)
- [ ] Existing tests still pass
- [ ] `pre-commit-check` passes
- [ ] Original repro steps no longer reproduce the bug (conceptually — this can't be verified end-to-end until the backend gap is also fixed, since triggering `credential_not_recognized` via admin host is the trigger condition itself)

## Completion Criteria

- [ ] Bug fixed
- [ ] Regression test added
- [ ] Existing tests pass
- [ ] Changes committed to `plan/bugfix-passkey-admin-fallback/task-03-defect-false-clear-on-admin-fallback` branch
- [ ] Status updated in `status.md`
- [ ] Blocked task unblocked (if applicable) — N/A, blocks nothing
