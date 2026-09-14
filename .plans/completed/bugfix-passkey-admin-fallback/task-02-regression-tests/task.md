# Task: Rewrite/add passkey login tests for the admin-host fallback

**Plan**: bugfix-passkey-admin-fallback
**Task ID**: task-02
**Task Path**: task-02-regression-tests
**Depends On**: task-01-admin-host-fallback
**Ticket**: N/A

## Objective

Rewrite the existing test that locks in the old (buggy) short-circuit behavior, and add regression coverage proving the new admin-host fallback works without breaking tenant-scoped passkey login.

## Context

Read [investigation.md](../investigation.md) Step 2c for full test coverage analysis. Summary:

- `passkey_login_cubit_test.dart:376-397` currently asserts the **old** behavior: empty subdomain → immediate `PasskeyLoginFailed`, `verifyNever(mockPasskeyRepository.getLoginChallenge())`. After task-01, this assertion is **wrong on purpose** — it must be rewritten, not just deleted, since it documents an important scenario (empty-subdomain passkey login) that still needs coverage, just with the new expected behavior.
- **Found during task-01 execution (see its status.md Adaptations)**: a second existing test also locks in the old behavior — `passkey_repository_impl_test.dart` → `"getLoginChallenge does NOT attempt to initialize NetworkService when no subdomain has ever been persisted (defensive — should be unreachable in practice...)"`. This one asserts `networkService.initService` is never called on an empty subdomain; after task-01 it now IS called (against the admin host). Rewrite this one too — same treatment as the cubit test, not just deletion.
- No existing test isolates `PasskeyRepositoryImpl._ensureNetworkServiceInitialized()`'s host-selection logic — add one.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch develop && git pull --rebase origin develop`
- [ ] Verify task-01-admin-host-fallback is complete (check its `status.md`)
- [ ] Check this task's `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read [investigation.md](../investigation.md) for full root cause context
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `test/features/authentication/passkey/application/passkey_login_cubit_test.dart` | modify | Rewrite lines 376-397; add fallback-path coverage |
| `test/features/authentication/passkey/infrastructure/passkey_repository_impl_test.dart` | modify | Add coverage for `_ensureNetworkServiceInitialized()`'s admin-host fallback |

### Do NOT Modify

- `lib/features/authentication/passkey/**` — owned by task-01-admin-host-fallback (implementation should already be complete and merged/available on this branch's base)

## Implementation Steps

### Step 1: Rewrite the empty-subdomain test in `passkey_login_cubit_test.dart`

Replace the `blocTest` at lines 376-397 (currently titled `'emits [InProgress, Failed] with the "no passkey" message ... when this device has never logged in on the current environment'`) so that with `AppSharedPreferences.setSubdomain('')`:
- The cubit **does** call `passkeyRepository.getLoginChallenge()` (remove/replace the `verifyNever` assertion — assert it **was** called instead)
- Simulate the mocked repository/native provider behaving as if hitting the admin host with no matching credential (`NativePasskeyNoCredentialException` or equivalent), and assert the flow still ends in the same friendly `PasskeyLoginFailed('No passkey was found on this device...')` message — but now reached through the real challenge/native-call path, not a short-circuit
- Update the test description to reflect the new intent (e.g., "falls through to the admin host and surfaces the same friendly no-credential message when nothing is enrolled there")

### Step 2: Add a "fallback succeeds" test

Add a new `blocTest` covering: empty subdomain, but the admin host *does* have a valid credential for this account (mocked challenge + successful native sign + verify) → asserts `PasskeyLoginSucceeded`, proving the fallback path isn't just a relabeled failure.

### Step 3: Add a "tenant subdomain unaffected" regression test (if not already covered)

Confirm there's an existing passing test with a persisted subdomain that still initializes against the tenant host, not the admin host. If none isolates this assertion explicitly, add one.

### Step 4: Cover `PasskeyRepositoryImpl._ensureNetworkServiceInitialized()` directly

In `passkey_repository_impl_test.dart`, add tests asserting:
- Empty subdomain → `networkService.initService` called with `environment.subdomainUrl('admin')`
- Non-empty subdomain → `networkService.initService` called with `environment.subdomainUrl(subdomain)` (existing behavior, must not regress)

## Testing

- [ ] New/rewritten tests fail against pre-task-01 code and pass against post-task-01 code (verify by temporarily checking out task-01's parent commit if needed, or reasoning from the diff)
- [ ] All existing tests pass
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB updates required for this task — covered under task-01's note on the broader passkey KB gap
- [ ] "No KB/doc updates required" — this task is test-only

## Completion Criteria

- [ ] Regression test (rewritten empty-subdomain test) fails before task-01's fix and passes after
- [ ] New fallback-success and tenant-unaffected tests pass
- [ ] All existing tests pass
- [ ] Documentation / KB updates completed or explicitly marked not needed
- [ ] Changes committed to `plan/bugfix-passkey-admin-fallback/task-02-regression-tests` branch
- [ ] Status updated in `status.md`
