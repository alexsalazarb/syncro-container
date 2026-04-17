# Task: Fix GetTicketsSettingsDeserializer FATAL crash

**Plan**: fix-production-crashes-v140
**Phase**: N/A (flat plan)
**Task ID**: task-01
**Task Path**: task-01-tickets-settings-fatal
**Depends On**: None
**JIRA**: N/A

**Priority**: HIGHEST — FATAL crash in Crashlytics (Android, appeared 3 days ago, FRESH signal)
**Crashlytics IDs**: `13ac72d53184ea74a3ffd6d26cbf3392` (FATAL), `793c17540aced95d629b218510cfc4a3` (NON_FATAL)

## Objective

When the backend returns `null` (or a non-Map) for the tickets-settings endpoint, the deserializer
calls `recordError(..., fatal: true)` then throws, creating a FATAL event in Crashlytics.
Change the behavior to log as NON_FATAL and return a graceful empty response instead of throwing.

## Context

**File**: `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart`

The deserializer currently:
1. Guards `data is! Map` → logs to Crashlytics with `fatal: true`
2. Throws `ArgumentError` → network service catches it and logs NON_FATAL as "Unexpected error in sendRequestCall"

Result: TWO Crashlytics issues per occurrence — one FATAL, one NON_FATAL — for a condition the network service already handles.

**Root cause**: `recordError(..., fatal: true)` marks the event as fatal when the app actually doesn't crash (the network service's `Either` wraps the failure gracefully).

**Fix**: When `data is! Map`, log as `fatal: false` and return `GetTicketsSettingsResponse(null)` instead of throwing. `GetTicketsSettingsResponse.ticketsSettings` is already declared nullable (`TicketsSettings?`), so callers already handle this case.

Before writing the fix, verify that the calling cubit handles `Right(GetTicketsSettingsResponse(null))` (i.e., `ticketsSettings == null`) without NPE. Search for the use site: grep for `GetTicketsSettingsResponse` in the cubit layer.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify `task-01-tickets-settings-fatal/status.md` is `not-started`
- [ ] Grep for `GetTicketsSettingsResponse` usages to confirm cubit handles null `ticketsSettings`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart` | modify | Change guard to return empty response + log NON_FATAL |
| `syncro-flutter/test/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer_test.dart` | create | New test file — none existed |

### Do NOT Modify

- `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` — owned by task-02
- `syncro-flutter/lib/core/services/chat_websocket_service.dart` — owned by task-03
- `syncro-flutter/lib/features/assets/domain/get_chat_information_by_ids_response.dart` — owned by task-04

## Implementation Steps

### Step 1: Verify null handling at the call site

```bash
cd syncro-flutter
grep -r "GetTicketsSettingsResponse" lib/ --include="*.dart" -l
```

Open the cubit that consumes this response and confirm it checks for `null` before accessing `ticketsSettings`. If the cubit crashes on null `ticketsSettings`, add a guard there first.

### Step 2: Fix the deserializer

In `get_tickets_settings_deserializer.dart`, replace the current `data is! Map` guard body:

```dart
// BEFORE
if (data is! Map) {
  final reason = 'GetTicketsSettingsDeserializer: expected Map, got ${data.runtimeType}';
  logger(reason, title: 'GetTicketsSettingsDeserializer');
  final error = ArgumentError(reason);
  FirebaseCrashlytics.instance.recordError(
    error, StackTrace.current, reason: reason, fatal: true,
  );
  throw error;
}

// AFTER
if (data is! Map) {
  final reason = 'GetTicketsSettingsDeserializer: expected Map, got ${data.runtimeType}';
  logger(reason, title: 'GetTicketsSettingsDeserializer');
  FirebaseCrashlytics.instance.recordError(
    ArgumentError(reason), StackTrace.current,
    reason: reason,
    fatal: false,  // Network service already handles this — not a real crash
  );
  return GetTicketsSettingsResponse(null);  // Graceful empty response
}
```

### Step 3: Write tests

Create `test/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer_test.dart`:

```dart
group('GetTicketsSettingsDeserializer', () {
  test('returns empty response when data is null', () {
    // should NOT throw
    final result = deserializer.fromJson(null);
    expect(result.ticketsSettings, isNull);
  });

  test('returns empty response when data is not a Map', () {
    final result = deserializer.fromJson([]);
    expect(result.ticketsSettings, isNull);
  });

  test('parses valid Map correctly', () {
    // use the minimal valid JSON from tickets_settings_test.dart
    final result = deserializer.fromJson(validJson);
    expect(result.ticketsSettings, isNotNull);
  });
});
```

Note: `FirebaseCrashlytics` must be mocked or the test will throw. Use `setUpAll` to initialize Firebase test mocks or run with `flutter test --dart-define=...`. Check how other tests in the project handle Crashlytics (see `test/` root for any `firebase_mock_setup.dart` or `setUp` patterns).

## Testing

- [ ] `null` data → returns `GetTicketsSettingsResponse(null)`, does not throw
- [ ] Non-Map data (e.g. `[]`, `"string"`) → returns empty response, does not throw
- [ ] Valid Map → parses and returns correct `GetTicketsSettingsResponse`
- [ ] No FATAL Crashlytics events logged when data is invalid (verify `fatal: false`)
- [ ] Existing `tickets_settings_test.dart` still passes
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required — implementation-only fix, pattern is not novel enough to document

## Completion Criteria

- [ ] Deserializer returns `GetTicketsSettingsResponse(null)` instead of throwing on null/non-Map input
- [ ] Crashlytics call uses `fatal: false`
- [ ] New test file covers null, non-Map, and valid-Map scenarios
- [ ] All existing tests pass
- [ ] Changes committed to `plan/fix-production-crashes-v140/task-01-tickets-settings-fatal` branch
- [ ] Status updated in `status.md`
