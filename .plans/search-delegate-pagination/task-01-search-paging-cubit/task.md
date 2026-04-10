# Task: SearchPagedResult Model + SearchPagingCubit

**Plan**: Search Delegate Pagination
**Phase**: 1
**Task ID**: task-01
**Task Path**: task-01-search-paging-cubit
**Depends On**: None
**JIRA**: N/A

## Objective

Create the `SearchPagedResult<T>` model, the `CustomSearchPagedCallback<T>` typedef, and the `SearchPagingCubit<T>` that owns a `PagingController<int, T>` and handles debounced query resets — the foundation for paginated search.

## Context

The existing `SearchCubit<T>` (`lib/core/delegates/search_cubit.dart`) fetches all results in one call via `CustomSearchCallback<T> = Future<List<T?>> Function(String query)`. The new paged variant replaces this contract with a per-page callback, and replaces the BLoC state machine with a `PagingController` (from `infinite_scroll_pagination`) that `PagedListView` listens to directly.

Reference the `AssetsCubit` pattern:
- `PagingController<int, Asset>` with `firstPageKey: 1`
- `pagingController.addPageRequestListener(...)` to trigger data fetches
- `pagingController.appendPage(items, nextPageKey)` / `appendLastPage(items)` / `.error = ...`
- `pagingController.refresh()` resets to page 1 (used on pull-to-refresh and query change)

The cubit's state itself is `void` — the `PagingController` is the reactive source of truth consumed by `PagedListView` directly without BLoC wiring.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read `syncro-flutter/lib/core/delegates/search_cubit.dart` — understand the existing debounce pattern
- [ ] Read `syncro-flutter/lib/features/assets/application/assets_cubit.dart` — the `PagingController` pattern to replicate
- [ ] Read `syncro-flutter/lib/core/utils/cubit_extension.dart` — use `safeEmit()` if any real states are emitted
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/delegates/search_paged_result.dart` | create | `SearchPagedResult<T>` + `CustomSearchPagedCallback<T>` typedef |
| `syncro-flutter/lib/core/delegates/search_paging_cubit.dart` | create | `SearchPagingCubit<T>` — owns `PagingController`, handles debounce |

### Do NOT Modify

- `syncro-flutter/lib/core/delegates/search_cubit.dart` — owned by existing code; no changes
- `syncro-flutter/lib/core/delegates/custom_search_delegate.dart` — owned by task-03

## Implementation Steps

### Step 1: Create `search_paged_result.dart`

Create `syncro-flutter/lib/core/delegates/search_paged_result.dart`:

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

/// Callback for paginated search operations.
/// Returns a [SearchPagedResult<T>] for the given [query] and [page].
typedef CustomSearchPagedCallback<T> =
    Future<SearchPagedResult<T>> Function(String query, int page);

/// Wraps a single page of search results.
class SearchPagedResult<T> {
  const SearchPagedResult({required this.items, required this.isLastPage});

  /// Items returned for this page.
  final List<T> items;

  /// Whether this is the final page (no more data to load).
  final bool isLastPage;
}
```

Note: `flutter_bloc` import is only needed if you add any Cubit base imports here; if not, remove it. Keep the file minimal — just the typedef and model.

### Step 2: Create `search_paging_cubit.dart`

Create `syncro-flutter/lib/core/delegates/search_paging_cubit.dart`:

```dart
import 'dart:async';

import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:infinite_scroll_pagination/infinite_scroll_pagination.dart';
import 'package:syncro/core/delegates/search_paged_result.dart';
import 'package:syncro/core/utils/logger.dart';

/// Cubit that owns a [PagingController] for paginated search.
///
/// State is managed entirely through [pagingController] — no Cubit states are
/// emitted. Consumers wire [pagingController] to a [PagedListView] directly.
class SearchPagingCubit<T> extends Cubit<void> {
  SearchPagingCubit({required this.pagedCallback}) : super(null) {
    pagingController.addPageRequestListener(_onPageRequested);
  }

  final CustomSearchPagedCallback<T> pagedCallback;

  /// The paging controller consumed by [PagedListView] in the delegate.
  final PagingController<int, T> pagingController =
      PagingController(firstPageKey: 1);

  String _currentQuery = '';
  Timer? _debounceTimer;

  /// Call when the search query changes. Debounces 300 ms, then resets the
  /// paging controller back to page 1 with the new query.
  void onQueryChanged(String query) {
    if (query == _currentQuery) return;
    _debounceTimer?.cancel();
    _debounceTimer = Timer(const Duration(milliseconds: 300), () {
      _currentQuery = query;
      // refresh() resets pagingController to firstPageKey (page 1).
      pagingController.refresh();
    });
  }

  Future<void> _onPageRequested(int pageKey) async {
    try {
      final result = await pagedCallback(_currentQuery, pageKey);
      if (result.isLastPage) {
        pagingController.appendLastPage(result.items);
      } else {
        pagingController.appendPage(result.items, pageKey + 1);
      }
    } catch (error) {
      logger('Page load error: $error', title: 'SearchPagingCubit');
      pagingController.error = error.toString();
    }
  }

  @override
  Future<void> close() {
    _debounceTimer?.cancel();
    pagingController.dispose();
    return super.close();
  }
}
```

**Key design notes:**
- `Cubit<void>` — no states emitted; `PagingController` is the reactive source
- `_currentQuery` captured at debounce-resolution time so in-flight page requests use the correct query
- `pagingController.refresh()` resets `itemList`, `error`, and page key to `firstPageKey`, then immediately triggers `_onPageRequested(1)`

## Testing

- [ ] Unit test: `onQueryChanged` with the same query twice does NOT reset `pagingController`
- [ ] Unit test: `onQueryChanged` with a different query resets `pagingController.value` (status = `loadingFirstPage`)
- [ ] Unit test: `_onPageRequested` success with `isLastPage: false` → `appendPage` called with `pageKey + 1`
- [ ] Unit test: `_onPageRequested` success with `isLastPage: true` → `appendLastPage` called
- [ ] Unit test: `_onPageRequested` throws → `pagingController.error` is set
- [ ] Unit test: `close()` cancels debounce timer and disposes `pagingController`
- [ ] Existing `SearchCubit` tests still pass
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc exists for the delegate/search pattern — if this task reveals non-obvious patterns, run `document-solution` after task-04 completes (the full picture will be clearer then)

## Completion Criteria

- [ ] `search_paged_result.dart` created with `SearchPagedResult<T>` and `CustomSearchPagedCallback<T>`
- [ ] `search_paging_cubit.dart` created with `SearchPagingCubit<T>`
- [ ] Unit tests cover the five scenarios listed above
- [ ] `flutter analyze` passes with no new warnings
- [ ] No regressions in existing `SearchCubit` behaviour
- [ ] Changes committed to `plan/search-delegate-pagination/task-01-search-paging-cubit` branch
- [ ] Status updated in `status.md`
