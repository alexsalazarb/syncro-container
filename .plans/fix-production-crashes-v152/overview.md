# Plan: Fix Production Non-Fatal Crashes v1.5.2

**Status**: not-started
**Created**: 2026-05-22
**Last Updated**: 2026-05-22
**Severity**: P2
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Base Branch**: main

## Objective

Fix 4 non-fatal production bugs detected in Crashlytics v1.5.2 (build 423). All bugs are currently OPEN. Combined impact: ~25 users across iOS and Android.

## Bug Summary

| # | Title | Platform | Events | Users | Severity |
|---|-------|----------|--------|-------|----------|
| 1 | Asset saved-searches deserializer type mismatch | iOS + Android | 20 | 11 | P2 |
| 2 | WebSocket null-check on `_connectingCompleter` after timeout | iOS | 11 | 11 | P2 |
| 3 | WorksheetTemplateDeserializer throws on empty paginated response | Android | 1 | 1 | P3 |
| 4 | 5xx responses recorded as Crashlytics non-fatals (noise) | iOS + Android | 14 | 8 | P3 |

## Root Causes

| Task | Root Cause |
|------|------------|
| task-01 | BE changed `GET /saved_searches` response from `List` to `Map<String, dynamic>` (pagination wrapper). Deserializer correctly detects the mismatch and records to Crashlytics, but should instead adapt to the new shape. |
| task-02 | `_connectingCompleter` can be set to null concurrently (e.g. `disconnect()` race) during the retry loop. After 3 failed attempts, line 648 uses `!` on a potentially-null completer → `Null check operator on null value`. |
| task-03 | BE added `metadata` key alongside `worksheet_templates`. Deserializer iterates all keys; when `worksheet_templates` exists but contains an empty list (0 results), `worksheets.isEmpty` triggers a `FormatException` — conflates "empty valid response" with "unrecognized format". |
| task-04 | `_handleFailure` in `RestNetworkServiceImpl` explicitly records all `statusCode >= 500` errors to Crashlytics. 502/504 are transient server-side gateway errors — not client bugs. Recording them creates noise and obscures real app issues. |

## Task Summary

| Task | Title | JIRA | Status | Depends On |
|------|-------|------|--------|------------|
| task-01-asset-filter-deserializer | Asset saved-searches: handle Map response | SE-12498 | not-started | — |
| task-02-websocket-null-completer | WebSocket: null-safe completer after timeout | SE-12499 | complete | — |
| task-03-worksheet-empty-response | Worksheet: empty paginated response | SE-12500 | complete | — |
| task-04-suppress-5xx-noise | Suppress 5xx errors from Crashlytics | SE-12501 | complete | — |

All tasks are independent — can be executed in parallel.

## Branch Convention

Pattern: `plan/fix-production-crashes-v152/{task-id}`

Examples:
- `plan/fix-production-crashes-v152/task-01-asset-filter-deserializer`
- `plan/fix-production-crashes-v152/task-02-websocket-null-completer`

Base branch: main

## Key Files

| File | Task |
|------|------|
| `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` | task-01 |
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | task-02 |
| `syncro-flutter/lib/features/ticket/ticket_details/worksheets/infrastructure/worksheet_deserializers.dart` | task-03 |
| `syncro-flutter/lib/core/networking/services/rest_api/rest_network_service.dart` | task-04 |

## Crashlytics Issue IDs

| Issue ID | Platform | Task |
|----------|----------|------|
| `3ca075b4171de3e9bed7700c513e575f` | iOS | task-01 |
| `9a1d349f325abb563d2b26653a1b993c` | Android | task-01 |
| `544ed9e78b79ddd91868924bcd1effcc` | iOS | task-02 |
| `5622a83782a7edb0ec3e7359c30a465c` | Android | task-03 |
| `7f75ca76ae6ea9853645ce230a735531` | iOS | task-04 |
| `7ec1fff22d998a861ff0d1705d05518d` | Android | task-04 |

## Success Criteria

- [ ] `GetAssetsFiltersSavedSearchesDeserializer` handles Map response without recording to Crashlytics
- [ ] `ChatWebSocketService.connect` does not throw when `_connectingCompleter` is null after timeout
- [ ] `WorksheetTemplateDeserializer` returns empty list (not throw) when `worksheet_templates` is present but empty
- [ ] `RestNetworkServiceImpl._handleFailure` does not record 502/503/504 responses to Crashlytics
- [ ] All 6 Crashlytics issues show zero new events after deploy
- [ ] All tests pass (`flutter test`)
- [ ] `flutter analyze` passes
