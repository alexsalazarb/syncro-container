# Plan: Search Delegate Pagination

**Status**: not-started
**Created**: 2026-04-09
**Last Updated**: 2026-04-09
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: N/A
**Base Branch**: main

## Objective

Add infinite-scroll pagination with pull-to-refresh and load-next-page-on-scroll to `CustomSearchDelegate<T>` and `CustomSearchCustomerOrAsset`, using `infinite_scroll_pagination` (`PagingController`, `PagedListView`, `PagedChildBuilderDelegate`) following the same pattern as `AssetsView`.

## Scope

### In Scope

- New `SearchPagedResult<T>` model wrapping items + `isLastPage` flag
- New `CustomSearchPagedCallback<T>` typedef: `Future<SearchPagedResult<T>> Function(String query, int page)`
- New `SearchPagingCubit<T>` owning a `PagingController<int, T>` with debounced query-reset behaviour
- Optional paginated mode in `CustomSearchDelegate<T>` — existing non-paginated call sites are unaffected
- `AssetsCubit.searchPaged()` method leveraging the existing `SearchAssetsParams.page` field and `Meta.totalPages`
- Wire `CustomSearchCustomerOrAsset` asset-search path to use the new paginated callback
- Pull-to-refresh via `CustomRefreshIndicator` wrapping the `PagedListView` in paged mode

### Out of Scope

- Migrating existing non-asset call sites (tickets, technicians, customers) to paginated mode — they retain the existing `customSearchCallback` path
- Customer search pagination (customers are typically a small set; scoped to asset search only)
- New UI empty/error widgets — reuses `CustomSearchEmptyStateWidget` and `CustomLoadingWidget`

## Kill Criteria

- If `SearchAssetsParams.page` is ignored by the API (server-side bug returning all results regardless) — pagination would silently duplicate items
- If `Meta.totalPages` is absent or always 1 in the search endpoint response — the `isLastPage` calculation would always terminate after page 1

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Foundation | task-01, task-02 | None | Paging cubit + assets search method (parallel) |
| 2 | Delegate | task-03 | task-01 | Paginated mode in CustomSearchDelegate |
| 3 | Integration | task-04 | task-02, task-03 | Wire CustomSearchCustomerOrAsset |

## Task Summary

| Task Path | Title | Phase | Status | Depends On |
|-----------|-------|-------|--------|------------|
| task-01-search-paging-cubit | SearchPagedResult + SearchPagingCubit | 1 | not-started | — |
| task-02-assets-cubit-paged-search | AssetsCubit.searchPaged() | 1 | not-started | — |
| task-03-delegate-paginated-mode | CustomSearchDelegate paginated mode | 2 | not-started | task-01-search-paging-cubit |
| task-04-wire-customer-or-asset | Wire CustomSearchCustomerOrAsset | 3 | not-started | task-02-assets-cubit-paged-search, task-03-delegate-paginated-mode |

## Branch Convention

Pattern: `plan/search-delegate-pagination/{task-path}`

Examples:
- `plan/search-delegate-pagination/task-01-search-paging-cubit`
- `plan/search-delegate-pagination/task-03-delegate-paginated-mode`

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/core/delegates/custom_search_delegate.dart` | Main search delegate to extend |
| `syncro-flutter/lib/core/delegates/search_cubit.dart` | Existing non-paginated cubit (reference) |
| `syncro-flutter/lib/core/delegates/custom_search_customer_or_asset.dart` | Orchestrates two-step customer→asset search |
| `syncro-flutter/lib/features/assets/application/assets_cubit.dart` | Owns `PagingController` pattern to follow |
| `syncro-flutter/lib/features/assets/presentation/assets_view.dart` | Reference: `PagedListView` + `CustomRefreshIndicator` usage |
| `syncro-flutter/lib/features/assets/domain/get_asset_params.dart` | `SearchAssetsParams` — includes optional `page` field |
| `syncro-flutter/lib/features/assets/domain/meta.dart` | `Meta.totalPages` — determines `isLastPage` |
| `syncro-flutter/lib/core/global_widgets/custom_refresh_indicator.dart` | Shared pull-to-refresh widget |

## Risks

- `PagingController` must be disposed **and re-created** (not just refreshed) when the delegate is closed — the cubit `close()` handles this, but the delegate must call `_searchPagingCubit.close()` in its own `dispose()`
- Query debounce must call `pagingController.refresh()` (reset to page 1) rather than appending — confirmed via `PagingController.refresh()` resetting to `firstPageKey`
- `CustomSearchDelegate` currently requires `customSearchCallback` as a positional required param — changing it to optional is a **breaking change at compile time**; add `assert` to guard against both being null

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] Searching in `CustomSearchCustomerOrAsset` asset mode loads page 1, scrolling to bottom loads page 2+
- [ ] Pulling down the search results list refreshes from page 1
- [ ] Query change resets pagination to page 1 (debounced 300 ms)
- [ ] All existing non-paginated call sites of `CustomSearchDelegate` compile and behave identically
- [ ] `flutter analyze` passes with no new warnings
- [ ] Unit tests for `SearchPagingCubit` cover: query change resets controller, page load success, page load error, last-page detection

## References

- **JIRA Epic**: N/A
- **Related Plans**: None
