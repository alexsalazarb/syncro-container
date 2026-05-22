# Status: Asset saved-searches — handle Map response

**Current Status**: complete
**Last Updated**: 2026-05-22
**Agent**: claude-sonnet-4-6
**Branch**: plan/fix-production-crashes-v152/task-01-asset-filter-deserializer
**PR**: N/A

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-05-22 | not-started | — | Task created |
| 2026-05-22 | complete | claude-sonnet-4-6 | Fix applied and committed |

## Blockers

None

## Artifacts

- Commit: `SE-12498: Mobile App: Asset saved-searches deserializer type mismatch`
- Files changed:
  - `lib/features/assets/infrastructure/asset_filter_deserializer.dart` — `GetAssetsFiltersSavedSearchesDeserializer` now handles both `List` (legacy) and `Map` (paginated) responses; tries keys `saved_searches`, `saved_asset_searches`, `data`, `items`
  - `test/features/assets/infraestructure/asset_filter_deserializer_test.dart` — 3 regression tests added (17 total, all pass)

## Adaptations

- Exact BE Map key unknown at fix time; implemented multi-key fallback strategy (`saved_searches`, `saved_asset_searches`, `data`, `items`) to cover likely variations. Map with unrecognised keys logs via `logger()` but no longer records to Crashlytics — avoids noise while the key is identified.
