# Task 04: Suppress transient 5xx errors from Crashlytics

**Plan**: fix-production-crashes-v152
**Phase**: —
**JIRA**: SE-12501
**Depends On**: —
**Crashlytics**: `7f75ca76ae6ea9853645ce230a735531` (iOS), `7ec1fff22d998a861ff0d1705d05518d` (Android)

## Objective

Stop recording 502, 503, and 504 HTTP errors as Crashlytics non-fatals in `RestNetworkServiceImpl._handleFailure`. These are transient server-side gateway errors that create noise and obscure real client bugs.

## Context

In `rest_network_service.dart:239-247`:
```dart
if (statusCode >= 500) {
  FirebaseCrashlytics.instance.recordError(
    e, e.stackTrace, fatal: false,
    reason: 'Server error $statusCode on ...',
  );
}
```

502 (Bad Gateway), 503 (Service Unavailable), and 504 (Gateway Timeout) are infrastructure/proxy issues. They self-resolve and are monitored by BE teams. Recording them in Crashlytics creates false noise for the mobile team. Only 500 (Internal Server Error) and 501 (Not Implemented) indicate persistent server bugs potentially worth tracking from the mobile side.

## Steps

1. **Change the condition** in `_handleFailure` from:
   ```dart
   if (statusCode >= 500) {
   ```
   to:
   ```dart
   if (statusCode == 500 || statusCode == 501) {
   ```

2. **Write regression test** — see Acceptance Criteria.

3. **Run** `fvm flutter test test/core/networking/` and `fvm flutter analyze`.

## File Ownership

**MAY Modify**:
- `syncro-flutter/lib/core/networking/services/rest_api/rest_network_service.dart` (`_handleFailure` method only)
- `syncro-flutter/test/core/networking/services/rest_api/rest_network_service_test.dart` (create or update)

**Do NOT Modify**:
- Any other file

## Acceptance Criteria

- [ ] `_handleFailure` does NOT call `recordError` for status codes 502, 503, 504
- [ ] `_handleFailure` STILL calls `recordError` for status code 500
- [ ] `_handleFailure` STILL calls `recordError` for status code 501
- [ ] Existing error handling/return values for all status codes are unchanged
- [ ] Test: 500 → `recordError` called
- [ ] Test: 502 → `recordError` NOT called
- [ ] Test: 503 → `recordError` NOT called
- [ ] Test: 504 → `recordError` NOT called
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
