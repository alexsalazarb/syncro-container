# Task: Wire CustomSearchCustomerOrAsset to Paginated Search

**Plan**: Search Delegate Pagination
**Phase**: 3
**Task ID**: task-04
**Task Path**: task-04-wire-customer-or-asset
**Depends On**: task-02-assets-cubit-paged-search, task-03-delegate-paginated-mode
**JIRA**: N/A

## Objective

Update `CustomSearchCustomerOrAsset` to pass `customSearchPagedCallback` (backed by `AssetsCubit.searchPaged()`) when opening the asset search, activating the paginated `PagedListView` path in `CustomSearchDelegate` for all asset search flows in the app.

## Context

`CustomSearchCustomerOrAsset` (`lib/core/delegates/custom_search_customer_or_asset.dart`) currently opens `CustomSearchDelegate` with `customSearchCallback: (query) => assetsCubit.search(query, customerId)`. After this task, it passes `customSearchPagedCallback: (query, page) => assetsCubit.searchPaged(query, page, customerId)` instead, enabling the paginated list view.

Two methods are affected:
1. `handleSearchWithCustomer()` — asset search scoped to a customer
2. `handleSearch<T>()` — the generic handler that both `handleSearchWithCustomer()` and external callers use

The customer-search path (`handleSearchCustomer<T>()`) is **not** paginated in this plan — customers are a small set. It remains on the existing `customSearchCallback` path.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify task-02 is complete: `AssetsCubit.searchPaged()` exists
- [ ] Verify task-03 is complete: `CustomSearchDelegate` accepts `customSearchPagedCallback`
- [ ] Read `custom_search_customer_or_asset.dart` in full — understand the two-step flow
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/delegates/custom_search_customer_or_asset.dart` | modify | Pass `customSearchPagedCallback` for asset search |

### Do NOT Modify

- `syncro-flutter/lib/features/assets/application/assets_cubit.dart` — owned by task-02 (already complete)
- `syncro-flutter/lib/core/delegates/custom_search_delegate.dart` — owned by task-03 (already complete)
- Any feature-level views that call `CustomSearchCustomerOrAsset` — no changes required there

## Implementation Steps

### Step 1: Update imports

Add to `custom_search_customer_or_asset.dart` (alongside the existing `CustomSearchDelegate` show import):

```dart
import 'package:syncro/core/delegates/custom_search_delegate.dart'
    show CustomSearchDelegate, CustomSearchCallback, CustomSearchPagedCallback, OnTrailingButtonPressed;
import 'package:syncro/core/delegates/search_paged_result.dart';
```

Ensure `CustomSearchPagedCallback` is re-exported from `custom_search_delegate.dart` or imported directly from `search_paged_result.dart`.

### Step 2: Update `handleSearch<T>()` signature

Add an optional `customSearchPagedCallback` parameter:

```dart
void handleSearch<T>({
  required BuildContext context,
  required String searchLabel,
  CustomSearchCallback? customSearchCallback,
  CustomSearchPagedCallback<T>? customSearchPagedCallback,
}) {
  assert(
    customSearchCallback != null || customSearchPagedCallback != null,
    'Provide either customSearchCallback or customSearchPagedCallback',
  );
  var result = showSearchCapitalize(
    context: context,
    delegate: CustomSearchDelegate<T>(
      context: context,
      searchLabel: searchLabel,
      preserveSearchOnStack: true,
      customSearchCallback: customSearchCallback,
      customSearchPagedCallback: customSearchPagedCallback,
      trailingButtonIcon: trailingButtonIcon,
      onTrailingButtonPressed: onTrailingButtonPressed,
    ),
  );
  result.then((value) {
    // ...existing switch logic unchanged...
  });
}
```

### Step 3: Update `handleSearchWithCustomer()` to use paginated callback

```dart
void handleSearchWithCustomer(
  BuildContext context,
  String searchLabel,
  int? customerId,
) {
  final assetsCubit = context.read<AssetsCubit>();
  handleSearch<Asset>(
    context: context,
    searchLabel: searchLabel,
    customSearchPagedCallback: (query, page) =>
        assetsCubit.searchPaged(query, page, customerId),
  );
}
```

Note: `customSearchCallback` is no longer passed here — `customSearchPagedCallback` activates the paginated delegate path.

### Step 4: Verify `handleSearchCustomer<T>()` is unchanged

This method searches customers (not assets) and should remain on the non-paginated path. Confirm it still passes `customSearchCallback:` to `CustomSearchDelegate` unchanged.

## Testing

- [ ] Manual test (device/simulator): open asset search from `AssetsPage` → scroll down → confirm page 2 loads
- [ ] Manual test: pull down in the search results list → confirm results reset to page 1
- [ ] Manual test: type a query → confirm results filter and reset to page 1 (debounced)
- [ ] Manual test: open asset search from a ticket (customer-scoped) → same paginated behaviour
- [ ] Regression: open customer search (`handleSearchCustomer`) → still shows full list (non-paginated, no regressions)
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] Run `document-solution` after this task completes — capture the full paginated search delegate pattern (SearchPagingCubit + CustomSearchDelegate paginated mode + AssetsCubit.searchPaged) as a project KB doc for future reference

## Completion Criteria

- [ ] `handleSearchWithCustomer()` uses `customSearchPagedCallback` backed by `AssetsCubit.searchPaged()`
- [ ] `handleSearch<T>()` accepts both callback types (either one may be provided)
- [ ] `handleSearchCustomer<T>()` is unchanged — still uses `customSearchCallback`
- [ ] Manual pagination and pull-to-refresh verified on device
- [ ] `flutter analyze` passes with no new warnings
- [ ] Changes committed to `plan/search-delegate-pagination/task-04-wire-customer-or-asset` branch
- [ ] Status updated in `status.md`
