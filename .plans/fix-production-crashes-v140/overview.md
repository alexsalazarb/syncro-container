# Plan: Fix Production Crashes — v1.4.0

**Status**: not-started
**Created**: 2026-04-16
**Last Updated**: 2026-04-20
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A (PR_INTEGRATION=false)
**Base Branch**: main

## Objective

Fix four production crashes reported in Firebase Crashlytics for Syncro Mobile v1.4.0,
covering a FATAL deserializer crash, two null-safety issues, and a null check race condition
in the WebSocket service. DioMixin 504 errors are excluded (handled by the backend team).

## Scope

### In Scope
- `GetTicketsSettingsDeserializer.fromJson` FATAL — returns graceful empty response when `data` is null/non-Map (Android FATAL: 3 events/1 user + NON_FATAL: 3 events/1 user)
- `GetAssetsFiltersSavedSearchesDeserializer.fromJson` — null guard + proper error propagation (iOS: 129 events/33 users, Android: 93 events/18 users — **Android escalando**)
- `ChatWebSocketService.connect` — capture `_socket` as local var to eliminate null-check race condition after `await` (iOS: 70 events/~54 users, Android: 6 events/5 users)
- `CustomerInformation.fromJson` — null-safe casts for `fullname` y `business_name` (iOS: 7 events/2 users, Android: 18 events/2 users)

### Out of Scope
- DioMixin 504 Gateway Timeout — backend team handling
- ANR issues (`MessageQueue.nativePollOnce`) — old versions 1.2.1–1.3.1, low frequency
- Refactoring the entire `ChatWebSocketService` reconnection strategy

## Kill Criteria

- If the backend changes the tickets-settings API contract (nullable response becomes impossible), task-01 becomes moot — verify before executing
- If `PhoenixSocket` upgrades its API between investigation and fix (breaking the local variable capture approach)

## Phases

No phases — all 4 tasks are independent and can run in parallel.

## Task Summary

| Task Path | Title | Status | Priority | Eventos totales | Usuarios totales |
|-----------|-------|--------|----------|-----------------|-----------------|
| task-01-tickets-settings-fatal | Fix GetTicketsSettingsDeserializer FATAL crash | not-started | **HIGHEST** — FATAL Android | 6 | 1 |
| task-02-asset-filter-deserializer | Fix GetAssetsFiltersSavedSearchesDeserializer | not-started | **HIGH** — mayor volumen, Android escalando | 222 | 51 |
| task-03-websocket-null-check | Fix ChatWebSocketService.connect null check | not-started | **HIGH** — ~59 usuarios afectados | 76 | ~59 |
| task-04-customer-info-null-cast | Fix CustomerInformation.fromJson null cast | not-started | MEDIUM | 25 | ~4 |

## Branch Convention

Pattern: `plan/fix-production-crashes-v140/{task-path}`

Examples:
- `plan/fix-production-crashes-v140/task-01-tickets-settings-fatal`
- `plan/fix-production-crashes-v140/task-02-asset-filter-deserializer`
- `plan/fix-production-crashes-v140/task-03-websocket-null-check`
- `plan/fix-production-crashes-v140/task-04-customer-info-null-cast`

Base branch: `main`

## Key Files

| File | Task |
|------|------|
| `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart` | task-01 |
| `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` | task-02 |
| `syncro-flutter/lib/features/assets/domain/assets_filters_data.dart` | task-02 (AssetFilter.fromJson) |
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | task-03 |
| `syncro-flutter/lib/features/assets/domain/get_chat_information_by_ids_response.dart` | task-04 |

## Firebase Crashlytics Issue IDs

| Issue | Platform | ID | Type |
|-------|----------|----|------|
| asset_filter_deserializer | iOS | 3ca075b4171de3e9bed7700c513e575f | NON_FATAL |
| asset_filter_deserializer | Android | 9a1d349f325abb563d2b26653a1b993c | NON_FATAL |
| chat_websocket connect | iOS | 544ed9e78b79ddd91868924bcd1effcc | NON_FATAL |
| chat_websocket connect | Android | a7bc3afa8e582c9aec48aee6663ca02d | NON_FATAL |
| customer_info fromJson | iOS | a28ba3f3b903eb994045b446987862ae | NON_FATAL |
| customer_info fromJson | Android | 0d9f870b504dd2a1c3124683ab253f71 | NON_FATAL |
| tickets_settings_deserializer | Android | 13ac72d53184ea74a3ffd6d26cbf3392 | **FATAL** |
| tickets_settings_deserializer | Android | 793c17540aced95d629b218510cfc4a3 | NON_FATAL |

## Risks

- `CustomerInformation` null fields: defaulting to `''` may expose empty strings in the UI — confirm UX is acceptable before shipping
- `GetTicketsSettingsResponse(null)` as graceful default: verify the cubit/repo handles a null `ticketsSettings` without NPE

## Success Criteria

- [ ] 0 new occurrences in Crashlytics for all 4 issues after releasing a patch
- [ ] All existing tests continue to pass
- [ ] New tests cover the null/invalid data scenarios that caused each crash
- [ ] No new `fatal: true` Crashlytics events from the deserializer layer
- [ ] `flutter analyze` passes with no new warnings

## References

- **JIRA Epic**: N/A
- **Related Plans**: None
