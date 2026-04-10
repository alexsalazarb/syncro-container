# Task: AssetsCubit.searchPaged() Method

**Plan**: Search Delegate Pagination
**Phase**: 1
**Task ID**: task-02
**Task Path**: task-02-assets-cubit-paged-search
**Depends On**: None
**JIRA**: N/A

## Objective

Add a `searchPaged(String query, int page, int? customerId)` method to `AssetsCubit` that returns `SearchPagedResult<Asset>`, enabling the search delegate to load assets page by page.

## Context

`AssetsCubit.search()` (`lib/features/assets/application/assets_cubit.dart`) currently fetches all results in one call (no `page` param). `SearchAssetsParams` already has an optional `page` field and `AssetsListResponse` returns `Meta` with `totalPages`, so the API already supports pagination. The new method mirrors the logic of `loadAssets()` but returns a `SearchPagedResult<Asset>` instead of feeding `pagingController`.

Key types:
- `SearchAssetsParams` — `lib/features/assets/domain/get_asset_params.dart` — accepts optional `page`, `query`, `customerId`
- `AssetsListResponse.meta?.totalPages` — from `Meta` — determines whether more pages exist
- `SearchPagedResult<T>` — created in task-01 (`lib/core/delegates/search_paged_result.dart`)

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read `syncro-flutter/lib/features/assets/application/assets_cubit.dart` — understand existing `search()` and `loadAssets()` methods
- [ ] Read `syncro-flutter/lib/features/assets/domain/get_asset_params.dart` — confirm `page` field exists on `SearchAssetsParams`
- [ ] Read `syncro-flutter/lib/features/assets/domain/meta.dart` — confirm `totalPages` field
- [ ] Verify task-01 is complete and `search_paged_result.dart` exists
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/assets/application/assets_cubit.dart` | modify | Add `searchPaged()` method |

### Do NOT Modify

- `syncro-flutter/lib/features/assets/domain/get_asset_params.dart` — existing `page` field is sufficient; do not add fields
- `syncro-flutter/lib/core/delegates/custom_search_customer_or_asset.dart` — owned by task-04
- `syncro-flutter/lib/core/delegates/search_paged_result.dart` — owned by task-01

## Implementation Steps

### Step 1: Add import to `assets_cubit.dart`

Add the import for `SearchPagedResult`:

```dart
import 'package:syncro/core/delegates/search_paged_result.dart';
```

### Step 2: Add `searchPaged()` method

Add the following method to `AssetsCubit`, after the existing `search()` method:

```dart
/// Fetches a single page of asset search results.
/// Returns [SearchPagedResult<Asset>] with [isLastPage] derived from
/// [Meta.totalPages]. On failure, returns an empty last page so the
/// paging controller terminates cleanly rather than showing an error.
Future<SearchPagedResult<Asset>> searchPaged(
  String query,
  int page,
  int? customerId,
) async {
  final params = SearchAssetsParams(
    query: query,
    customerId: customerId,
    page: page,
  );
  final result = await searchAssetsUseCase.call(params);
  return result.fold(
    (failure) {
      logger('searchPaged error: ${failure.message}', title: 'AssetsCubit');
      return SearchPagedResult<Asset>(items: const [], isLastPage: true);
    },
    (response) {
      final assets =
          (response.assets ?? []).whereType<Asset>().toList();
      final totalPages = response.meta?.totalPages ?? 1;
      return SearchPagedResult<Asset>(
        items: assets,
        isLastPage: page >= totalPages,
      );
    },
  );
}
```

**Design notes:**
- On API failure, returns an empty last page instead of throwing — this prevents the `PagingController` from showing an error state in the search overlay, consistent with how `search()` returns `[]` on failure
- Uses `whereType<Asset>()` to match the null-filtering pattern in `loadAssets()`
- `isLastPage: page >= totalPages` handles edge cases where the API returns `totalPages: 0`

## Testing

- [ ] Unit test: success path returns correct `items` and `isLastPage: false` when `page < totalPages`
- [ ] Unit test: success path returns `isLastPage: true` when `page >= totalPages`
- [ ] Unit test: failure path returns `SearchPagedResult(items: [], isLastPage: true)`
- [ ] Existing `AssetsCubit` tests still pass (no regressions)
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No standalone KB update needed — the pattern will be documented in `document-solution` after task-04 completes

## Completion Criteria

- [ ] `searchPaged()` method added to `AssetsCubit`
- [ ] Unit tests cover success (mid-pages), success (last page), and failure scenarios
- [ ] `flutter analyze` passes with no new warnings
- [ ] No regressions in existing `search()` or `loadAssets()` behaviour
- [ ] Changes committed to `plan/search-delegate-pagination/task-02-assets-cubit-paged-search` branch
- [ ] Status updated in `status.md`
