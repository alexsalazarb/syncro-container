# Status: Defect — passkey login never learns the real subdomain, breaking the post-login `/me` call

**Current Status**: not-started
**Last Updated**: 2026-09-14
**Agent**: N/A
**Branch**: `plan/bugfix-passkey-admin-fallback/task-04-defect-post-login-subdomain-gap` (not yet created)
**PR**: N/A (PR_INTEGRATION=false)

## Status History

| Timestamp | Status | Notes |
|-----------|--------|-------|
| 2026-09-14 | not-started | Defect task created — found during real-device staging verification on `ss1`, right after backend MR 19414's global-credential-lookup fix (commit `ac8ce8`) made passkey sign-in itself start succeeding. Backend has already shipped its half of the fix (commit `07477a6d`, `subdomain` field on the login-verify response, deployed and confirmed live on `ss1`) — this task is entirely client-side. |

## Blockers

None — backend prerequisite (MR 19414 commit `07477a6d`) is already deployed and confirmed live on `ss1`.

## Artifacts

None yet.

## Adaptations

None yet.
