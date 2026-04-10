# Task: CustomSearchDelegate Paginated Mode

**Plan**: Search Delegate Pagination
**Phase**: 2
**Task ID**: task-03
**Task Path**: task-03-delegate-paginated-mode
**Depends On**: task-01-search-paging-cubit
**JIRA**: N/A

## Objective

Extend `CustomSearchDelegate<T>` with an optional paginated mode. When `customSearchPagedCallback` is supplied, the delegate uses `SearchPagingCubit<T>` and renders a `PagedListView` with `CustomRefreshIndicator`; when absent, the existing `customSearchCallback` / `SearchCubit<T>` path is used unchanged — preserving backward compatibility for all ~15 existing call sites.

## Context

`CustomSearchDelegate<T>` (`lib/core/delegates/custom_search_delegate.dart`) currently requires `customSearchCallback` and uses `SearchCubit<T>`. The goal is to make `customSearchCallback` optional and add `customSearchPagedCallback` as a second optional, with an assert ensuring exactly one is provided.

In paginated mode:
- Construct `SearchPagingCubit<T>` in the constructor
- `_onQueryChanged` forwards to `_searchPagingCubit!.onQueryChanged(query)` instead of `_searchCubit!.search(query)`
- `buildResultsAndSuggestions` returns `_buildPagedResultsList()` instead of the BLoC builder
- `dispose()` calls `_searchPagingCubit?.close()` in addition to `_searchCubit?.close()`
- `_buildPagedResultsList()` mirrors `AssetsView`: `CustomRefreshIndicator` > `PagedListView` > `PagedChildBuilderDelegate` — reuses the existing `_buildItemTile()` logic extracted from `_buildItemsList()`

Reference:
- `syncro-flutter/lib/features/assets/presentation/assets_view.dart` — `PagedListView` + `CustomRefreshIndicator` exact pattern
- `syncro-flutter/lib/core/global_widgets/custom_refresh_indicator.dart` — pull-to-refresh widget
- `syncro-flutter/lib/core/global_widgets/custom_search_empty_state.dart` — existing empty state widget
- `syncro-flutter/lib/core/global_widgets/global_widgets.dart` — `CustomLoadingWidget`

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-01 is complete: `search_paged_result.dart` and `search_paging_cubit.dart` exist
- [ ] Read the full `custom_search_delegate.dart` — understand all constructor params, state machine, and `_buildItemsList()`
- [ ] Read `assets_view.dart` — copy the `PagedListView` / `CustomRefreshIndicator` pattern
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/delegates/custom_search_delegate.dart` | modify | Add paginated mode (optional param + new builder) |

### Do NOT Modify

- `syncro-flutter/lib/core/delegates/search_paging_cubit.dart` — owned by task-01
- `syncro-flutter/lib/core/delegates/search_cubit.dart` — unchanged
- `syncro-flutter/lib/core/delegates/custom_search_customer_or_asset.dart` — owned by task-04

## Implementation Steps

### Step 1: Add imports

Add to `custom_search_delegate.dart`:

```dart
import 'package:infinite_scroll_pagination/infinite_scroll_pagination.dart';
import 'package:syncro/core/delegates/search_paged_result.dart';
import 'package:syncro/core/delegates/search_paging_cubit.dart';
import 'package:syncro/core/global_widgets/custom_refresh_indicator.dart';
```

### Step 2: Update constructor and fields

Make `customSearchCallback` optional; add `customSearchPagedCallback`; guard with assert:

```dart
class CustomSearchDelegate<T> extends CapitalizeSearchDelegate {
  CustomSearchDelegate({
    required this.context,
    required this.searchLabel,
    this.customSearchCallback,          // now optional
    this.customSearchPagedCallback,     // new optional
    this.trailingButtonIcon,
    this.onTrailingButtonPressed,
    this.onEditButtonPressed,
    this.showLeading = false,
    this.preserveSearchOnStack = false,
  }) : super() {
    assert(
      customSearchCallback != null || customSearchPagedCallback != null,
      'Provide either customSearchCallback or customSearchPagedCallback',
    );
    if (customSearchPagedCallback != null) {
      _searchPagingCubit = SearchPagingCubit<T>(
        pagedCallback: customSearchPagedCallback!,
      );
    } else {
      _searchCubit = SearchCubit<T>(
        searchCallback: customSearchCallback!,
        initialItems: initialItems,
      );
    }
  }

  // ...existing fields...
  final CustomSearchCallback? customSearchCallback;       // was required, now optional
  final CustomSearchPagedCallback<T>? customSearchPagedCallback; // new

  SearchCubit<T>? _searchCubit;
  SearchPagingCubit<T>? _searchPagingCubit;
  // ...rest unchanged...
}
```

### Step 3: Update `_onQueryChanged`

```dart
void _onQueryChanged(String query) {
  if (query != _lastQuery) {
    _lastQuery = query;
    if (_searchPagingCubit != null) {
      _searchPagingCubit!.onQueryChanged(query);
    } else {
      _searchCubit!.search(query);
    }
  }
}
```

### Step 4: Update `dispose()`

```dart
@override
void dispose() {
  _searchCubit?.close();
  _searchPagingCubit?.close();
  super.dispose();
}
```

### Step 5: Extract item tile builder

Extract the `ListTile` from `_buildItemsList()` into a private `_buildItemTile(T item, List<T> items, int index)` method so both `ListView` and `PagedListView` can reuse it without duplication.

### Step 6: Add `_buildPagedResultsList()`

```dart
/// Builds a paginated list with pull-to-refresh.
Widget _buildPagedResultsList() {
  return CustomRefreshIndicator(
    onRefresh: () async =>
        _searchPagingCubit!.pagingController.refresh(),
    child: PagedListView<int, T>(
      pagingController: _searchPagingCubit!.pagingController,
      builderDelegate: PagedChildBuilderDelegate<T>(
        firstPageProgressIndicatorBuilder: (_) =>
            const CustomLoadingWidget(),
        newPageProgressIndicatorBuilder: (_) =>
            const CustomLoadingWidget(),
        noItemsFoundIndicatorBuilder: (_) =>
            const CustomSearchEmptyStateWidget(),
        firstPageErrorIndicatorBuilder: (_) =>
            const CustomSearchEmptyStateWidget(),
        newPageErrorIndicatorBuilder: (_) =>
            const CustomSearchEmptyStateWidget(),
        itemBuilder: (ctx, item, index) => _buildItemTile(item, index),
      ),
    ),
  );
}
```

### Step 7: Update `buildResultsAndSuggestions()`

```dart
Widget buildResultsAndSuggestions(BuildContext context) {
  if (_searchPagingCubit != null) {
    return _buildPagedResultsList();
  }
  return BlocProvider.value(
    value: _searchCubit!,
    child: BlocBuilder<SearchCubit<T>, SearchState<T>>(
      builder: (context, state) {
        // ...existing switch unchanged...
      },
    ),
  );
}
```

## Testing

- [ ] Widget test: `CustomSearchDelegate` constructed with only `customSearchCallback` — renders existing `BlocBuilder` path (no regression)
- [ ] Widget test: `CustomSearchDelegate` constructed with only `customSearchPagedCallback` — renders `PagedListView`
- [ ] Widget test: pull-to-refresh triggers `pagingController.refresh()`
- [ ] Assert fires when both callbacks are null (debug mode)
- [ ] All 15+ existing call sites compile without changes (they pass `customSearchCallback:`, which still works)
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] After task-04 completes, run `document-solution` to capture the full paginated-search-delegate pattern in the project KB

## Completion Criteria

- [ ] `customSearchCallback` is now optional; `customSearchPagedCallback` added
- [ ] Paginated mode renders `PagedListView` wrapped in `CustomRefreshIndicator`
- [ ] Non-paginated mode behaviour is identical to pre-change
- [ ] `dispose()` closes both cubit references safely
- [ ] Widget tests pass for both modes
- [ ] `flutter analyze` passes with no new warnings
- [ ] Changes committed to `plan/search-delegate-pagination/task-03-delegate-paginated-mode` branch
- [ ] Status updated in `status.md`
