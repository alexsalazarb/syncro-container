# Task: Fix CustomerInformation.fromJson null cast

**Plan**: fix-production-crashes-v140
**Phase**: N/A (flat plan)
**Task ID**: task-04
**Task Path**: task-04-customer-info-null-cast
**Depends On**: None
**JIRA**: N/A

**Priority**: MEDIUM — iOS: 7 events/2 users, Android: 18 events/2 users | actualizado 2026-04-20
**Crashlytics IDs**: `a28ba3f3b903eb994045b446987862ae` (iOS), `0d9f870b504dd2a1c3124683ab253f71` (Android)
**Error**: "type 'Null' is not a subtype of type 'String' in type cast"

## Objective

Fix `CustomerInformation.fromJson` in `get_chat_information_by_ids_response.dart`.
The fields `fullname` and `business_name` are cast directly to `String`, but the backend
can return `null` for either field. Add null-safe coercion to prevent the type cast crash.

## Context

**File**: `syncro-flutter/lib/features/assets/domain/get_chat_information_by_ids_response.dart`

**Current crash site**:
```dart
factory CustomerInformation.fromJson(Map<String, dynamic> json) {
  return CustomerInformation(
    id: json['id'] as int,
    fullname: json['fullname'] as String,          // CRASH: Null is not a subtype of String
    businessName: json['business_name'] as String, // CRASH: Null is not a subtype of String
  );
}
```

**Fix**: Coerce null to empty string:
```dart
fullname: (json['fullname'] as String?) ?? '',
businessName: (json['business_name'] as String?) ?? '',
```

Before shipping, confirm the UI that displays `fullname` and `businessName` handles empty
strings gracefully (no layout overflow or misleading "Unknown" labels expected). This is
a data-layer fix only — no UI changes in scope unless a regression is found.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Verify `task-04-customer-info-null-cast/status.md` is `not-started`
- [ ] Grep for `CustomerInformation` usages in the presentation layer to confirm empty strings display acceptably
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/assets/domain/get_chat_information_by_ids_response.dart` | modify | Null-safe `fullname` and `business_name` |
| `syncro-flutter/test/features/assets/domain/get_chat_information_by_ids_response_test.dart` | create | New test file — none existed |

### Do NOT Modify

- `syncro-flutter/lib/features/ticket/ticket_home/infrastructure/get_tickets_settings_deserializer.dart` — owned by task-01
- `syncro-flutter/lib/features/assets/infrastructure/asset_filter_deserializer.dart` — owned by task-02
- `syncro-flutter/lib/core/services/chat_websocket_service.dart` — owned by task-03

## Implementation Steps

### Step 1: Fix `CustomerInformation.fromJson`

In `get_chat_information_by_ids_response.dart`, line 46–52:

```dart
// BEFORE
factory CustomerInformation.fromJson(Map<String, dynamic> json) {
  return CustomerInformation(
    id: json['id'] as int,
    fullname: json['fullname'] as String,
    businessName: json['business_name'] as String,
  );
}

// AFTER
factory CustomerInformation.fromJson(Map<String, dynamic> json) {
  return CustomerInformation(
    id: json['id'] as int,
    fullname: (json['fullname'] as String?) ?? '',
    businessName: (json['business_name'] as String?) ?? '',
  );
}
```

### Step 2: Review `AssetInformation.fromJson` in the same file

Check `json['name'] as String` on line 30 — apply the same null-safe pattern if `name` can also be null:
```dart
name: (json['name'] as String?) ?? '',
```

Also check `json['id'] as int` — if `id` can be null, default to `0` or throw (ID being null is a more serious data issue).

### Step 3: Write tests

Create `test/features/assets/domain/get_chat_information_by_ids_response_test.dart`:

```dart
group('CustomerInformation.fromJson', () {
  test('parses valid json correctly', () {
    final json = {'id': 1, 'fullname': 'John Doe', 'business_name': 'Acme'};
    final result = CustomerInformation.fromJson(json);
    expect(result.fullname, 'John Doe');
    expect(result.businessName, 'Acme');
  });

  test('defaults null fullname to empty string', () {
    final json = {'id': 1, 'fullname': null, 'business_name': 'Acme'};
    final result = CustomerInformation.fromJson(json);
    expect(result.fullname, '');
  });

  test('defaults null business_name to empty string', () {
    final json = {'id': 1, 'fullname': 'John', 'business_name': null};
    final result = CustomerInformation.fromJson(json);
    expect(result.businessName, '');
  });

  test('defaults both fields to empty string when null', () {
    final json = {'id': 1, 'fullname': null, 'business_name': null};
    final result = CustomerInformation.fromJson(json);
    expect(result.fullname, '');
    expect(result.businessName, '');
  });
});
```

## Testing

- [ ] `null` `fullname` → defaults to `''`, no crash
- [ ] `null` `business_name` → defaults to `''`, no crash
- [ ] Both null → both `''`, no crash
- [ ] Valid data → parsed correctly
- [ ] `fvm fvm flutter test test/features/assets/domain/` passes
- [ ] `fvm fvm flutter analyze` passes

## Documentation / KB Updates

- [ ] No KB doc updates required

## Completion Criteria

- [ ] `fullname` and `business_name` are null-safe in `CustomerInformation.fromJson`
- [ ] `AssetInformation.name` reviewed and fixed if nullable
- [ ] New test file with 4+ null-safety scenarios
- [ ] All existing tests pass
- [ ] Changes committed to `plan/fix-production-crashes-v140/task-04-customer-info-null-cast` branch
- [ ] Status updated in `status.md`
