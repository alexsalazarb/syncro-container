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

Justin's (backend) confirmation that the WebAuthn RP ID is genuinely the shared `admin.$basePath` host per environment (see overview.md Kill Criteria) had **not landed** as of this task starting. Alex explicitly decided (2026-09-10, asked directly and confirmed "todavía no respondió, arrancamos igual") to start implementation anyway, accepting the risk that the Kill Criteria assumption could be disconfirmed later and require a revert. This was a deliberate, informed choice — not an oversight. Still open; staging verification (overview.md Completion Checklist) still requires it before this can be considered fully done end-to-end.

## Artifacts

- `lib/features/authentication/passkey/infrastructure/passkey_repository_impl.dart` — `_ensureNetworkServiceInitialized()` now falls back to `environment.subdomainUrl('admin')` when no subdomain is persisted
- `lib/features/authentication/passkey/application/passkey_login_cubit.dart` — removed the empty-subdomain short-circuit in `signIn()`; always attempts `getLoginChallenge()` now

## Adaptations

Found a **second** pre-existing test locking in the old behavior, beyond the one already flagged in investigation.md: `passkey_repository_impl_test.dart` — `"getLoginChallenge does NOT attempt to initialize NetworkService when no subdomain has ever been persisted (defensive — should be unreachable in practice...)"`. This test now fails for the same reason (by design) as `passkey_login_cubit_test.dart`'s equivalent test. Not a scope change to this task (task-01 only owns implementation files) — flagged forward to task-02, which already owns rewriting `passkey_repository_impl_test.dart` per its file ownership table; added an explicit step there so it isn't missed.
