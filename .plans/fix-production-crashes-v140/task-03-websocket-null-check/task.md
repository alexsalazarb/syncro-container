# Task: Fix ChatWebSocketService.connect null check race condition

**Plan**: fix-production-crashes-v140
**Phase**: N/A (flat plan)
**Task ID**: task-03
**Task Path**: task-03-websocket-null-check
**Depends On**: None
**JIRA**: N/A

**Priority**: HIGH — 57 unique iOS users affected, 3 Android users
**Crashlytics IDs**: `544ed9e78b79ddd91868924bcd1effcc` (iOS), `a7bc3afa8e582c9aec48aee6663ca02d` (Android)
**Error**: "Null check operator used on a null value" in `ChatWebSocketService.connect`

## Objective

Eliminate the null check operator (`!`) race condition in `ChatWebSocketService.connect()`.
After an `await`, another async path (e.g. `disconnect()`) can set `_socket = null`,
causing the subsequent `_socket!.isConnected` to crash. Fix by capturing `_socket` in a
local variable at the start of `connect()` so it can't be nulled by concurrent code.

## Context

**File**: `syncro-flutter/lib/core/services/chat_websocket_service.dart`

**Root cause**: `connect()` uses `_socket!` after `await` calls. In Dart, `await` suspends
the isolate and returns control to the event loop. During that suspension, `disconnect()`
(which sets `_socket = null`) can execute. When `connect()` resumes, `_socket` is null
and `_socket!` throws.

Relevant lines in `connect()`:
```dart
if (_socket == null) { return false; }     // null check here...
if (_socket!.isConnected) { return true; } // ...but ! used here (synchronous — safe)

// Inside the retry loop:
await _socket!.connect().timeout(...)      // AWAIT — gives up control
if (_socket!.isConnected) { ... }         // After await, _socket could be null → CRASH
```

The synchronous `_socket!.isConnected` on line 534 (before any `await`) is technically
safe in single-threaded Dart. The crash happens on `_socket!.isConnected` at line 570,
which comes AFTER `await _socket!.connect()`.

**Fix**: Capture `_socket` as a local variable `final socket = _socket;` at the top of
`connect()`, null-check it once, then use `socket` (non-nullable local) throughout. A local
variable cannot be set to null by external code after capture.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify `task-03-websocket-null-check/status.md` is `not-started`
- [ ] Read `chat_websocket_service.dart` lines 520–616 (`connect()` method) carefully before editing
- [ ] Grep for all `_socket!` usages in `connect()` — replace every one with `socket`
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/core/services/chat_websocket_service.dart` | modify | Capture `_socket` as local var in `connect()` |
| `syncro-flutter/test/core/services/chat_websocket_service_connect_test.dart` | create | New test file — no tests existed for this service |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart` — owned by task-01
- `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` — owned by task-02
- `syncro-flutter/lib/features/assets/domain/get_chat_information_by_ids_response.dart` — owned by task-04

## Implementation Steps

### Step 1: Refactor `connect()` to use a local variable

In `chat_websocket_service.dart`, at the start of `connect()`, replace:

```dart
Future<bool> connect({int maxRetries = 3}) async {
  if (_connectingCompleter != null) { ... }

  if (_socket == null) {
    logger('🔌 Socket not initialised.');
    return false;
  }

  if (_socket!.isConnected) {   // safe — no await yet, but still better as local
    ...
  }
  ...
  for (int attempt = 1; attempt <= maxRetries && _shouldReconnect; attempt++) {
    try {
      await _socket!.connect().timeout(...)   // PROBLEM: after any await, _socket could be null
      if (_socket!.isConnected) { ... }       // CRASH HERE
```

With:

```dart
Future<bool> connect({int maxRetries = 3}) async {
  if (_connectingCompleter != null) { ... }

  final socket = _socket;   // Capture once — immune to concurrent nulling
  if (socket == null) {
    logger('🔌 Socket not initialised.');
    return false;
  }

  if (socket.isConnected) {
    logger('✅ Socket already connected.');
    return true;
  }
  ...
  for (int attempt = 1; attempt <= maxRetries && _shouldReconnect; attempt++) {
    try {
      await socket.connect().timeout(...)    // No ! needed — socket is non-nullable local
      if (socket.isConnected) { ... }        // Safe after await
```

Replace every remaining `_socket!` within `connect()` with `socket`. The field `_socket` (the class-level nullable) should not be accessed with `!` anywhere inside `connect()` after this change.

### Step 2: Review other `_socket!` usages in the file

Run `grep -n '_socket!' lib/core/services/chat_websocket_service.dart` and check each site:
- Inside other methods (e.g. `_tryInitialConnection`, `_createAndJoinChannel`) — apply the same local-variable pattern where an `await` precedes the `!` usage
- The `_socket?.addChannel(...)` calls are already null-safe (using `?.`) — leave them

Only fix methods where `_socket!` follows an `await` without a re-check of `_socket != null`.

### Step 3: Write targeted tests

`ChatWebSocketService` uses `GetIt` singletons and `PhoenixSocket` — both are hard to unit test without mocking. Create focused tests for the null-safety behavior using a simple approach:

```dart
// test/core/services/chat_websocket_service_connect_test.dart
group('ChatWebSocketService.connect', () {
  test('returns false when socket is not initialised', () async {
    // Service starts with _socket = null
    final service = ChatWebSocketService();
    // Call connect() directly — should return false without crashing
    // Note: singleton — may need a way to reset state for testing
  });
});
```

If the singleton pattern makes unit testing impractical, document WHY in the test file and skip — write an integration note instead. Do NOT add a testing hook or refactor the singleton just for this task.

## Testing

- [ ] `connect()` called when `_socket` is null → returns `false`, no crash
- [ ] `connect()` called when `_socket` is non-null → proceeds normally
- [ ] Simulated scenario: `disconnect()` called concurrently during `connect()` → no null check crash (document if not automatable)
- [ ] `flutter test` passes (all existing tests)
- [ ] `flutter analyze` passes

## Documentation / KB Updates

- [ ] If the singleton pattern prevents automated testing, add a note to `docs/kb-projects/syncro-flutter/technical/integrations/phoenix-socket.md` explaining the null-capture pattern for future maintainers

## Completion Criteria

- [ ] All `_socket!` usages inside `connect()` replaced with local `socket` variable
- [ ] Other `_socket!` sites following `await` in the file also fixed
- [ ] Test file created (or documented reason for skip)
- [ ] All existing tests pass
- [ ] `flutter analyze` passes with no new warnings
- [ ] Changes committed to `plan/fix-production-crashes-v140/task-03-websocket-null-check` branch
- [ ] Status updated in `status.md`
