# Investigation: fix-production-crashes-v152

**Investigated**: 2026-05-22
**Confidence**: High
**Source**: Crashlytics v1.5.2 (build 423) — last 7 days

---

## Bug 1 — Asset Saved-Searches Deserializer Type Mismatch

**Crashlytics IDs**: `3ca075b4171de3e9bed7700c513e575f` (iOS), `9a1d349f325abb563d2b26653a1b993c` (Android)
**Impact**: 20 events, 11 users — highest priority

### Stack Trace
```
GetAssetsFiltersSavedSearchesDeserializer.fromJson (asset_filter_deserializer.dart:31)
RestNetworkServiceImpl.sendRequestCall (rest_network_service.dart:184)
GetAssetsFiltersUseCase._getSavedSearches (get_assets_filters_usecase.dart:43)
GetAssetsFiltersUseCase.call (get_assets_filters_usecase.dart:20)
AssetFilterCubit.refreshFilters.<fn> (asset_filter_cubit.dart:27)
```

### Root Cause
`GetAssetsFiltersSavedSearchesDeserializer.fromJson` at line 26 checks:
```dart
if (data == null || data is! List) {
  _recordError(ArgumentError('...expected List, got _Map<String, dynamic>'), ...);
  return GetAssetFilterResponse([]);
}
```

The BE changed `GET /saved_searches` (or equivalent) response from a bare `List` to a `Map<String, dynamic>` — likely wrapping in a pagination envelope. The current code correctly detects this as a format mismatch and records to Crashlytics. The fix is to extract the list from the map instead of treating it as an error.

**Unknown**: the exact key name containing the list in the new Map response (e.g. `data`, `saved_searches`, `items`). The fix task must inspect the actual response or BE documentation before writing the adaptation logic.

### Trigger
User opens Assets tab → `AssetFilterCubit.refreshFilters()` fires immediately on screen load.

---

## Bug 2 — WebSocket Null Completer After Timeout

**Crashlytics ID**: `544ed9e78b79ddd91868924bcd1effcc` (iOS only)
**Impact**: 11 events, 11 users (1 crash per user)

### Stack Trace
```
ChatWebSocketService.connect (chat_websocket_service.dart:648)
ChatWebSocketService._tryInitialConnection (chat_websocket_service.dart:317)
ChatWebSocketService.getInteractions (chat_websocket_service.dart:673)
```

### Logs from Event
```
Socket connect error: TimeoutException: Connection timeout (10 s)
Socket failed after 3 attempts.
```

### Root Cause
`connect()` at line 593 assigns `_connectingCompleter = Completer<bool>()`. After 3 failed attempts (timeout), line 648 executes:
```dart
_connectingCompleter!.complete(false);  // ← throws if null
```

A concurrent call to `disconnect()` (which nulls `_connectingCompleter`) can race with the retry loop during `await Future.delayed(delay)` at line 639. When the `!` operator fires after the delay, `_connectingCompleter` is already null → `Null check operator used on a null value`.

**Note**: A similar fix was already applied for `_socket` (see comment at line 561-565, referencing this same Crashlytics issue ID). The `_socket` race was fixed by capturing a local variable. `_connectingCompleter` was missed.

**Fix**: Change line 648 to `_connectingCompleter?.complete(false)`.

### Why iOS Only
Likely related to iOS low-battery / unstable network behavior causing more WebSocket timeouts. Android users may experience the same conditions less frequently. Previous Crashlytics session (SE-12379 hotfix investigation) found similar iOS-specific WebSocket instability patterns.

---

## Bug 3 — WorksheetTemplateDeserializer Empty Paginated Response

**Crashlytics ID**: `5622a83782a7edb0ec3e7359c30a465c` (Android)
**Impact**: 1 event, 1 user — `firstSeenVersion: 1.5.1` (regression)

### Stack Trace
```
WorksheetTemplateDeserializer.fromJson (worksheet_deserializers.dart:135)
RestNetworkServiceImpl.sendRequestCall (rest_network_service.dart:184)
WorksheetRepositoryImpl.getWorksheetTemplates (worksheet_repository_impl.dart:25)
TicketDetailsCubit.getWorksheetTemplates (ticket_details_cubit.dart:299)
SearchCubit._performSearch (search_cubit.dart:57)
```

### Error
```
FormatException: Invalid worksheet response format. Expected worksheet_templates,
worksheet_results, or direct worksheet object. Received keys: [worksheet_templates, metadata].
```

### Root Cause
The BE added a `metadata` key to the worksheet response (pagination). The deserializer at line 134:
```dart
if (worksheets.isEmpty) {
  throw FormatException('Invalid worksheet response format... Received keys: ${data.keys.toList()}');
}
```

When the response is `{worksheet_templates: [], metadata: {page: 1, total: 0}}` — an empty paginated result — the `worksheet_templates` key is present but contains an empty list. The parsing loop adds 0 worksheets. Then `worksheets.isEmpty` fires and throws a `FormatException`, treating a valid empty response as a format error.

**Fix**: Track whether a recognized key (`worksheet_templates`, `worksheet_results`) was found. Only throw if NO recognized key was found. An empty list under a recognized key is a valid response.

---

## Bug 4 — 5xx Responses Recorded to Crashlytics

**Crashlytics IDs**: `7f75ca76ae6ea9853645ce230a735531` (iOS), `7ec1fff22d998a861ff0d1705d05518d` (Android)
**Impact**: 14 events, 8 users

### Stack Trace
```
DioMixin.request (dio_mixin.dart:364)
RestNetworkServiceImpl._performRequest (rest_network_service.dart:206)
RestNetworkServiceImpl.sendRequestCall (rest_network_service.dart:173)
TicketDetailsCubit.getTicketById (ticket_details_cubit.dart:107)
```

### Root Cause
`_handleFailure` in `rest_network_service.dart:234-247`:
```dart
if (statusCode >= 500) {
  FirebaseCrashlytics.instance.recordError(e, e.stackTrace, fatal: false,
    reason: 'Server error $statusCode on ...');
}
```

This intentionally records ALL 5xx errors. 502 (Bad Gateway) and 504 (Gateway Timeout) are transient infrastructure issues, not app bugs — they resolve themselves and are not actionable by the mobile team. Recording them in Crashlytics creates noise that obscures real client-side bugs.

**Fix**: Exclude 502, 503, 504 from Crashlytics recording. Only report 500 and 501 (which may indicate real server bugs worth tracking). Alternatively, remove 5xx reporting entirely since BE teams have their own monitoring.
