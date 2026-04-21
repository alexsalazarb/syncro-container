# Task: Fix GetAssetsFiltersSavedSearchesDeserializer crashes

**Plan**: fix-production-crashes-v140
**Phase**: N/A (flat plan)
**Task ID**: task-02
**Task Path**: task-02-asset-filter-deserializer
**Depends On**: None
**JIRA**: N/A

**Priority**: HIGH — mayor volumen total (iOS: 129 events/33 users, Android: 93 events/18 users — Android escalando ↑) | actualizado 2026-04-20
**Crashlytics IDs**: `3ca075b4171de3e9bed7700c513e575f` (iOS), `9a1d349f325abb563d2b26653a1b993c` (Android)

## Objective

Fix two root causes in `asset_filter_deserializer.dart`:
1. Missing null/type guard on `data` before casting to `List<dynamic>`
2. `catch (e) { throw Exception(); }` swallows the original error — Crashlytics never sees the real cause

Also fix `AssetFilter.fromJson` — it accesses `json['id']` and `json['name']` without null guards.

## Context

**Files**:
- `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart`
- `syncro-flutter/lib/features/assets/domain/assets_filters_data.dart` (`AssetFilter.fromJson`)

**Current `GetAssetsFiltersSavedSearchesDeserializer.fromJson`**:
```dart
GetAssetFilterResponse fromJson(dynamic data) {
  try {
    final filtersData = data as List<dynamic>;       // Crashes if data is null or Map
    List<AssetFilter> assetFilters = filtersData
        .map((assetFilter) => AssetFilter.fromJson(assetFilter))  // Can throw per-item
        .toList();
    return GetAssetFilterResponse(assetFilters);
  } catch (e) {
    throw Exception();   // Swallows root cause — Crashlytics only sees "Unexpected error in sendRequestCall"
  }
}
```

**Current `AssetFilter.fromJson`**:
```dart
factory AssetFilter.fromJson(Map<String, dynamic> json) {
  return AssetFilter(
    id: json['id'],     // int? → int — NPE if null
    name: json['name'], // String? → String — NPE if null
  );
}
```

**Fix strategy**:
- In `GetAssetsFiltersSavedSearchesDeserializer.fromJson`: guard `data` for null/non-List, log real error to Crashlytics, return empty `GetAssetFilterResponse([])` instead of re-throwing
- In `AssetFilter.fromJson`: add null-safe coercion for `id` and `name`
- Apply the same null guard fix to `GetAssetsFiltersAssetTypesDeserializer.fromJson` (`data['asset_types']` can also be null)

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify `task-02-asset-filter-deserializer/status.md` is `not-started`
- [ ] Read `test/features/assets/infraestructure/asset_filter_deserializer_test.dart` — existing tests verify `throwsException` on bad input; after the fix the behavior changes to return empty, so tests MUST be updated
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` | modify | Add null guards, log real errors, return empty response |
| `syncro-flutter/lib/features/assets/domain/assets_filters_data.dart` | modify | Null-safe `AssetFilter.fromJson` |
| `syncro-flutter/test/features/assets/infraestructure/asset_filter_deserializer_test.dart` | modify | Update existing tests to reflect new behavior |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart` — owned by task-01
- `syncro-flutter/lib/core/services/chat_websocket_service.dart` — owned by task-03
- `syncro-flutter/lib/features/assets/domain/get_chat_information_by_ids_response.dart` — owned by task-04

## Implementation Steps

### Step 1: Fix `AssetFilter.fromJson`

In `assets_filters_data.dart`, make `id` and `name` null-safe:

```dart
factory AssetFilter.fromJson(Map<String, dynamic> json) {
  return AssetFilter(
    id: (json['id'] as int?) ?? 0,
    name: (json['name'] as String?) ?? '',
  );
}
```

### Step 2: Fix `GetAssetsFiltersSavedSearchesDeserializer.fromJson`

```dart
@override
GetAssetFilterResponse fromJson(dynamic data) {
  if (data == null || data is! List) {
    FirebaseCrashlytics.instance.recordError(
      ArgumentError('GetAssetsFiltersSavedSearchesDeserializer: expected List, got ${data?.runtimeType}'),
      StackTrace.current,
      reason: 'Unexpected data type in asset saved-searches response',
      fatal: false,
    );
    return GetAssetFilterResponse([]);
  }
  try {
    return GetAssetFilterResponse(
      (data as List<dynamic>)
          .map((item) => AssetFilter.fromJson(item as Map<String, dynamic>))
          .toList(),
    );
  } catch (e, stackTrace) {
    FirebaseCrashlytics.instance.recordError(
      e, stackTrace,
      reason: 'GetAssetsFiltersSavedSearchesDeserializer.fromJson parse error',
      fatal: false,
    );
    return GetAssetFilterResponse([]);
  }
}
```

### Step 3: Fix `GetAssetsFiltersAssetTypesDeserializer.fromJson`

Apply the same null guard to `data['asset_types']`:

```dart
@override
GetAssetFilterResponse fromJson(dynamic data) {
  if (data == null || data is! Map) {
    FirebaseCrashlytics.instance.recordError(
      ArgumentError('GetAssetsFiltersAssetTypesDeserializer: expected Map, got ${data?.runtimeType}'),
      StackTrace.current,
      reason: 'Unexpected data type in asset-types response',
      fatal: false,
    );
    return GetAssetFilterResponse([]);
  }
  final assetTypes = data['asset_types'];
  if (assetTypes == null || assetTypes is! List) {
    FirebaseCrashlytics.instance.recordError(
      ArgumentError('GetAssetsFiltersAssetTypesDeserializer: asset_types missing or not a List'),
      StackTrace.current,
      reason: 'asset_types field missing in asset-types response',
      fatal: false,
    );
    return GetAssetFilterResponse([]);
  }
  try {
    return GetAssetFilterResponse(
      (assetTypes as List<dynamic>)
          .map((item) => AssetFilter.fromJson(item as Map<String, dynamic>))
          .toList(),
    );
  } catch (e, stackTrace) {
    FirebaseCrashlytics.instance.recordError(
      e, stackTrace,
      reason: 'GetAssetsFiltersAssetTypesDeserializer.fromJson parse error',
      fatal: false,
    );
    return GetAssetFilterResponse([]);
  }
}
```

### Step 4: Update existing tests

In `test/features/assets/infraestructure/asset_filter_deserializer_test.dart`:

- Change `'fromJson throws Exception on invalid data'` → `'fromJson returns empty response on null data'` and `'fromJson returns empty response on non-List data'`
- Add test: `'fromJson handles individual item with null fields gracefully'`
- Verify the happy-path tests still pass

## Testing

- [ ] `null` data → returns `GetAssetFilterResponse([])`, does not throw
- [ ] Non-List data (String, Map) → returns empty response, does not throw
- [ ] Item with null `id`/`name` → coerces to `0`/`''`, does not throw
- [ ] Valid list → returns correct filters
- [ ] `GetAssetsFiltersAssetTypesDeserializer` with missing `asset_types` key → returns empty
- [ ] All updated tests pass: `fvm fvm flutter test test/features/assets/infraestructure/`
- [ ] `fvm fvm flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required

## Completion Criteria

- [ ] Both deserializers guard null/non-List input and return empty `GetAssetFilterResponse`
- [ ] `AssetFilter.fromJson` handles null `id`/`name` without throwing
- [ ] Real errors are logged to Crashlytics with `fatal: false` (not swallowed)
- [ ] Existing test file updated — no test expects `throwsException` for null/invalid input
- [ ] All tests pass
- [ ] Changes committed to `plan/fix-production-crashes-v140/task-02-asset-filter-deserializer` branch
- [ ] Status updated in `status.md`
