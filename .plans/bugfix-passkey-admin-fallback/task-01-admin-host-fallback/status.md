# Status: Fall back to admin host when no subdomain is persisted

**Current Status**: complete
**Last Updated**: 2026-09-10
**Agent**: Claude (execute-task)
**Branch**: `plan/bugfix-passkey-admin-fallback/task-01-admin-host-fallback`
**PR**: N/A (PR_INTEGRATION=false — merges directly to `develop`)

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-09-10 | not-started | Task created |
| 2026-09-10 | in-progress | Branch created off `develop`; implementation started |
| 2026-09-10 | complete | Fix implemented, `flutter analyze` clean, `pre-commit-check` clean. 2 pre-existing tests now fail by design (owned by task-02) |

## Blockers

~~Justin's (backend) confirmation...~~ — **RESOLVED 2026-09-10**: Justin confirmed ("Yes 👍 It should be. If it's not working, let me know.") after implementation had already started on Alex's explicit informed risk-acceptance (see history above). Kill Criteria item closed. Staging verification (real-device end-to-end test) still in progress — see task-02/investigation follow-up on the `credential_not_recognized` result from the first real-device attempt, being re-tested with a controlled enroll → fresh reinstall sequence.

## Artifacts

- `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` — `_ensureNetworkServiceInitialized()` now falls back to `environment.subdomainUrl('admin')` when no subdomain is persisted
- `lib/features/authentication/passkey/application/passkey_login_cubit.dart` — removed the empty-subdomain short-circuit in `signIn()`; always attempts `getLoginChallenge()` now

## Adaptations

Found a **second** pre-existing test locking in the old behavior, beyond the one already flagged in investigation.md: `passkey_repository_impl_test.dart` — `"getLoginChallenge does NOT attempt to initialize NetworkService when no subdomain has ever been persisted (defensive — should be unreachable in practice...)"`. This test now fails for the same reason (by design) as `passkey_login_cubit_test.dart`'s equivalent test. Not a scope change to this task (task-01 only owns implementation files) — flagged forward to task-02, which already owns rewriting `passkey_repository_impl_test.dart` per its file ownership table; added an explicit step there so it isn't missed.
