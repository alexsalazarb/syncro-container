# Task 01: Asset saved-searches — handle Map response

**Plan**: fix-production-crashes-v152
**Phase**: —
**JIRA**: —
**Depends On**: —
**Crashlytics**: `3ca075b4171de3e9bed7700c513e575f` (iOS), `9a1d349f325abb563d2b26653a1b993c` (Android)

## Objective

Fix `GetAssetsFiltersSavedSearchesDeserializer` to handle a `Map<String, dynamic>` response from the BE (new pagination wrapper) instead of recording a Crashlytics error and returning empty.

## Context

The BE changed `GET /saved_searches` response from a bare `List` to a `Map` with the list nested under a key. The deserializer at line 26 detects this mismatch and records to Crashlytics — correct behavior for an unknown format, but now we need to adapt.

**Unknown at plan creation**: the exact wrapper key (e.g. `data`, `saved_searches`, `items`). Must inspect the actual BE response before writing the fix.

## Steps

1. **Inspect the actual BE response format** — run the app or check BE docs/source to find what key holds the list in the new Map response. Look for `data`, `saved_searches`, `items`, `results`, or similar.

2. **Update `GetAssetsFiltersSavedSearchesDeserializer.fromJson`** at `asset_filter_deserializer.dart`:
   - If `data is Map`, try to extract the list from the identified key
   - Only call `_recordError` if the map doesn't contain the expected key (truly unrecognized format)
   - Keep the existing `_recordError` path as fallback for genuinely unknown formats

3. **Write regression test** — see Acceptance Criteria.

4. **Run** `fvm flutter test test/features/assets/` and `fvm flutter analyze`.

## File Ownership

**MAY Modify**:
- `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart`
- `syncro-flutter/test/features/assets/infrastructure/asset_filter_deserializer_test.dart` (create if absent)

**Do NOT Modify**:
- `syncro-flutter/lib/features/assets/domain/assets_filters_data.dart`
- `syncro-flutter/lib/features/assets/domain/get_asset_filter_response.dart`
- `syncro-flutter/lib/features/assets/application/asset_filter_cubit/asset_filter_cubit.dart`
- Any other file

## Acceptance Criteria

- [ ] `GetAssetsFiltersSavedSearchesDeserializer.fromJson` correctly parses the new Map response and returns the filters list
- [ ] `_recordError` is NOT called when data is a Map with the expected wrapper key
- [ ] Existing behavior (bare List) still works
- [ ] Test: `fromJson(List)` → returns filters (existing behavior)
- [ ] Test: `fromJson(Map with wrapper key)` → returns filters (new behavior, no Crashlytics call)
- [ ] Test: `fromJson(Map without expected key)` → records error, returns empty (unknown format)
- [ ] Test: `fromJson(null)` → records error, returns empty (unchanged)
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
