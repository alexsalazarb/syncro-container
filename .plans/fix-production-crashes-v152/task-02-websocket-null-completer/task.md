# Task 02: WebSocket — null-safe completer after timeout

**Plan**: fix-production-crashes-v152
**Phase**: —
**JIRA**: —
**Depends On**: —
**Crashlytics**: `544ed9e78b79ddd91868924bcd1effcc` (iOS)

## Objective

Fix `ChatWebSocketService.connect` to not throw `Null check operator used on a null value` when `_connectingCompleter` is null after the retry loop exhausts all attempts.

## Context

At line 648 of `chat_websocket_service.dart`:
```dart
_connectingCompleter!.complete(false);
```

`_connectingCompleter` can be set to null by a concurrent `disconnect()` call during the `await Future.delayed(delay)` retry pause. After the retry loop, the `!` operator throws. A similar race was already fixed for `_socket` at lines 561-565 (same file, same Crashlytics issue family). `_connectingCompleter` was missed.

## Steps

1. **Change line 648** in `chat_websocket_service.dart` from:
   ```dart
   _connectingCompleter!.complete(false);
   ```
   to:
   ```dart
   _connectingCompleter?.complete(false);
   ```

2. **Write regression test** — see Acceptance Criteria.

3. **Run** `fvm flutter test test/` and `fvm flutter analyze`.

## File Ownership

**MAY Modify**:
- `syncro-flutter/lib/core/services/chat_websocket_service.dart` (line 648 only)
- `syncro-flutter/test/core/services/chat_websocket_service_test.dart` (create or update)

**Do NOT Modify**:
- Any other file

## Acceptance Criteria

- [ ] Line 648 uses `?.complete(false)` instead of `!.complete(false)`
- [ ] Test: simulates 3 failed connection attempts followed by `disconnect()` mid-retry — no exception thrown
- [ ] Test: `connect()` returns `false` after exhausting retries (completer null case)
- [ ] Existing connect/disconnect tests still pass
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
