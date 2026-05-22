# Status: Suppress transient 5xx errors from Crashlytics

**Current Status**: complete
**Last Updated**: 2026-05-22
**Agent**: claude-sonnet-4-6
**Branch**: plan/fix-production-crashes-v152/task-04-suppress-5xx-noise
**PR**: N/A

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-05-22 | not-started | — | Task created |
| 2026-05-22 | complete | claude-sonnet-4-6 | Fix applied and committed |

## Blockers

None

## Artifacts

- Commit: `SE-12501: Mobile App: Remove transient 5xx (502/503/504) errors from Crashlytics non-fatal reporting`
- Files changed:
  - `lib/core/networking/services/rest_api/rest_network_service.dart` — condition changed from `statusCode >= 500` to `statusCode == 500 || statusCode == 501`
  - `test/core/networking/services/rest_api/rest_network_service_5xx_test.dart` — documentation test with testing rationale

## Adaptations

- Documentation test file used (no executable tests) — same pattern as `chat_websocket_service_connect_test.dart`. `FirebaseCrashlytics.instance` is a static singleton with no injection seam; verifying `recordError` calls requires invasive refactoring out of scope for this bug fix.
