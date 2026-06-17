# Investigation: SE-12760 — iOS Chat List Not Sorted by Most Recent

**Date**: 2026-06-16 → 2026-06-17 (updated)
**Investigator**: Alex Salazar + Claude Code
**Confidence**: High

## Summary

The iOS chat list sorts chats incorrectly for at least one customer account (SA 75758, hkge.syncromsp.com). The sort algorithm `_sortChats()` is mathematically correct, and it IS called on every WebSocket event. **The primary mobile-side bug is a race condition: `_createListeners()` in `ChatWebSocketService` accumulates duplicate subscriptions when called more than once, causing `_handleChatUpdate` to fire twice per event.** This explains the visible flicker (list reorders immediately after appearing). A secondary fragility exists in `ChatFromSocket.fromJson` which crashes on null `last_message`. If the final sort order is still wrong after fixing the race condition, a server-side issue with `inserted_at` values is the remaining cause.

---

## Step 2a: Origin Analysis

### `_sortChats()` — introduced in SCLP-815

```
commit e5b36419 — SCLP-815: Mobile App: Improve health check on ChatWebSocket service
```

`_sortChats()` predates SE-12379. SE-12379 did NOT introduce the sort — it introduced `lastMessageInsertedAt` field and `.whereType<Map>()` guard.

### SE-12379 — what it actually introduced

```
commit 135c1b79 — SE-12379 Syncro Mobile App - Unable to View Initiated or Existing Chats on iOS
```

- Added `.whereType<Map<String, dynamic>>()` to `ListChatsFromSocket.fromJson` — filters `total_count` integer from payload
- Added `lastMessageInsertedAt: DateTime.parse('${json['last_message']['inserted_at']}Z')` to `ChatFromSocket.fromJson`
- Added `_sortChats()` call in `_handleChatUpdate()` at line 222

Before SE-12379, `lastMessageInsertedAt` field did not exist, and `_sortChats()` existed but had no real `inserted_at` data to sort on. The sort may have been dormant or based on a different field.

**SE-12760 is therefore a regression introduced by SE-12379's `lastMessageInsertedAt` parsing** — sorting now uses a field that may not correlate with web app order for all accounts.

---

## Step 2b: Feature History — Chat Sort Timeline

| Commit | Change | Sort Impact |
|--------|--------|-------------|
| SCLP-815 | `_sortChats()` introduced | Sort exists but `lastMessageInsertedAt` not yet the primary key |
| SE-12379 (`135c1b79`) | `lastMessageInsertedAt` field added; sort now uses it | Sort now depends on server's `last_message.inserted_at` |
| SE-12379 fix (`c77080b5`) | Chats with missing asset info are skipped | Potential ordering side effect — fewer chats = sort result changes |

---

## Step 2c: Why Existing Tests Missed This

The `list_chats_from_socket_test.dart` tests verify parsing succeeds with well-formed data. There are no tests that:

1. Assert sort order with multiple chats and interleaved timestamps
2. Verify behavior when `last_message` is null or missing
3. Compare mobile sort output against the web app's expected order

The sort regression is invisible to unit tests because the tests don't exercise the interplay between `ListChatsFromSocket.fromJson` and `_sortChats()` end-to-end.

---

## Step 2d: Cross-Project Assessment

**Single-project bug** — The fix is entirely within `syncro-flutter`. No API contract changes are needed. If task-01 confirms the server sends wrong `inserted_at` timestamps, a separate backend ticket should be filed (out of mobile scope).

---

## Detailed Findings

### Finding 1: `_sortChats()` is correct

```dart
// chat_cubit.dart:260
List<Chat> _sortChats(List<Chat> chats) {
  final sortedChats = List<Chat>.from(chats)
    ..sort(
      (b, a) => b.lastMessageInsertedAt.compareTo(a.lastMessageInsertedAt),
    );
  return sortedChats.reversed.toList();
}
```

The parameters are confusingly named `(b, a)` but:
- `b.compareTo(a)` on DateTime: negative when b < a (b is older) → b placed before a → ascending
- `.reversed.toList()` → descending (newest first) ✓

Mathematically equivalent to `(a, b) => b.compareTo(a)`.

**NOT the cause** of the ordering bug. Cleanup is warranted for readability (task-03) but won't change behavior.

### Finding 2: `_handleChatUpdate()` always calls `_sortChats()`

```dart
// chat_cubit.dart:222
safeEmit(ChatLoaded(chats: _sortChats(chatsWithAssetInfo)));
```

Sort is applied after every WebSocket push. The sort happens on the correct state. **NOT the cause.**

### Finding 3: `ChatFromSocket.fromJson` — unsafe `last_message` access

```dart
// chat_websocket_service.dart:1169
lastMessageInsertedAt: DateTime.parse(
  '${(json['last_message'] as Map<String, dynamic>)['inserted_at']}Z',
),
```

Two failure modes:
1. `json['last_message'] == null` → `TypeError: Null is not a subtype of Map<String, dynamic>`
2. `json['last_message'] == {}` → `inserted_at` is null → `DateTime.parse("nullZ")` → `FormatException`

Either throws inside `.map(ChatFromSocket.fromJson).toList()` in `ListChatsFromSocket.fromJson`, which has no per-item try-catch. **This would crash the entire list parse** and emit no chats.

However, for SE-12760 chats ARE shown (just wrongly ordered), so this is not the direct cause — but it is a latent crash risk for accounts with empty/deleted chats. **Fix defensively in task-02.**

### Finding 4: `inserted_at` timezone handling

```dart
DateTime.parse('${json['last_message']['inserted_at']}Z')
```

Appends `Z` to force UTC. The regression test (`list_chats_from_socket_test.dart:9`) uses `'2024-01-01T12:00:00'` (no timezone), confirming the server sends naive timestamps. This is correct IF the server always sends UTC values without zone info.

If some accounts have `inserted_at` values with a zone offset already appended (e.g. `"2026-06-15T10:00:00-07:00"`), adding `Z` would produce `"2026-06-15T10:00:00-07:00Z"` → `FormatException`. This would also crash the list parse rather than silently reorder — so it can be ruled out for SE-12760.

### Finding 5: `ListChatsFromSocket.fromJson` — no ordering guarantee from server

```dart
factory ListChatsFromSocket.fromJson(Map<String, dynamic> json) {
  return ListChatsFromSocket(
    chats: json.values
        .whereType<Map<String, dynamic>>()
        .map(ChatFromSocket.fromJson)
        .toList(),
  );
}
```

The payload is a JSON object (`{id: chat, id: chat, ...}`). JSON objects are unordered at the protocol level. Dart preserves insertion order in `Map`, but the server may serialize interactions in any order. This is why client-side `_sortChats()` exists. If `_sortChats()` receives correct `inserted_at` timestamps, the output is correct regardless of server payload order.

### Finding 6: Likely remaining root cause (to be confirmed by task-01)

The most likely explanation for wrong order WITH correct sort logic and correct sort call:

**`last_message.inserted_at` from `interactions_batched` does not equal the "last activity" timestamp the web app sorts by.**

Possible reasons:
- Web app sorts by `last_message.created_at` (client receive time); socket uses `inserted_at` (DB write time)
- Archived/re-opened chats in certain accounts have `inserted_at` reset to the archive/reopen time
- The `interactions_batched` channel returns interactions for a specific scope that may differ from the web app's REST `/chat/all` endpoint sorting logic

**This is unconfirmable without seeing the actual payload for account SA 75758.** Task-01 adds diagnostic logging to capture this.

---

### Finding 7: Duplicate listeners — confirmed root cause of flicker

`ChatWebSocketService._createListeners()` subscribes to two streams but **never stores the returned `StreamSubscription` objects**:

```dart
void _createListeners() {
  _socket?.messageStream.listen(/* handler */);   // subscription not stored
  _channel!.messages.listen(/* handler */);       // subscription not stored
}
```

`_createListeners()` is called inside `_tryInitialConnection()`. This function is invoked:
1. From `ChatWebSocketService.init()` (via `route_cubit.dart:97` — **unawaited**, fire-and-forget)
2. From `getInteractions()` fallback: when `!connected || !_isInitialized || !(_channel?.canPush ?? false)` evaluates true

**The race**: `unawaited(chatWebSocketService.init())` fires (route_cubit.dart:97) while navigation to Home/Chat tab proceeds. Before `init()` completes, `ChatCubit.getChats()` calls `getInteractions()`, which finds `_isInitialized = false` and calls `_tryInitialConnection()` again. Both paths call `_createListeners()`, adding a second set of subscriptions to the channel messages stream. No previous subscriptions are cancelled.

**Result**: Every `interactions_batched` event fires BOTH listeners → two `_handleAddNewChat()` calls → two `chatsSocketController.add()` → two `_handleChatUpdate()` executions with two concurrent `getChatInformationByIdsUseCase` calls → different `chatsWithAssetInfo` subsets (race on async response timing) → two different `ChatLoaded` emissions → visible flicker.

**Fix**: Store both subscriptions in `StreamSubscription?` fields; cancel them at the top of `_createListeners()` before re-subscribing. See task-05.

---

## Conclusion

| Hypothesis | Status |
|-----------|--------|
| `_sortChats()` double-reversal bug | ❌ Ruled out — mathematically correct |
| `_handleChatUpdate()` missing sort call | ❌ Ruled out — already present at line 222 |
| `ChatFromSocket.fromJson` null crash | ⚠️ Latent risk — not the sorting bug but fragile |
| Duplicate `_createListeners()` race condition | ✅ **Confirmed** — causes flicker, double state emit |
| Server sends wrong `inserted_at` timestamps | ⚠️ Possible secondary cause — confirm after fixing race condition |
| iOS/Android DateTime parsing difference | ❌ Ruled out — `Z` appended consistently |

**Fix priority**: task-05 (duplicate listeners) → task-02 (null safety) → task-03 (sort readability) → task-04 (regression tests). If sort order is still wrong after task-05, file a backend ticket for incorrect `inserted_at` values on account SA 75758.
