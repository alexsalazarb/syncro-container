# Task 03: Worksheet — empty paginated response

**Plan**: fix-production-crashes-v152
**Phase**: —
**JIRA**: SE-12500
**Depends On**: —
**Crashlytics**: `5622a83782a7edb0ec3e7359c30a465c` (Android)

## Objective

Fix `WorksheetTemplateDeserializer.fromJson` to return an empty list (not throw) when the BE returns a valid paginated response with `worksheet_templates: []` plus a `metadata` key.

## Context

At line 134 of `worksheet_deserializers.dart`:
```dart
if (worksheets.isEmpty) {
  throw FormatException(
    'Invalid worksheet response format... Received keys: ${data.keys.toList()}',
  );
}
```

When the BE returns `{worksheet_templates: [], metadata: {page: 1, total: 0}}`, the `worksheet_templates` key IS present and recognized, but the list is empty — a valid paginated "no results" response. The `worksheets.isEmpty` check conflates this with a truly unrecognized format and throws.

## Steps

1. **Add a `recognizedKeyFound` flag** in `WorksheetTemplateDeserializer.fromJson` before the key iteration loop. Set it to `true` when `worksheet_templates`, `worksheet_results`, or a direct worksheet key is found.

2. **Change the `worksheets.isEmpty` guard** at line 134 to only throw when `!recognizedKeyFound`. When a recognized key was found but the list was empty, return `[]` silently.

3. **Write regression test** — see Acceptance Criteria.

4. **Run** `fvm flutter test test/features/ticket/` and `fvm flutter analyze`.

## File Ownership

**MAY Modify**:
- `syncro-flutter/lib/features/ticket/ticket_details/worksheets/infrastructure/worksheet_deserializers.dart`
- `syncro-flutter/test/features/ticket/ticket_details/worksheets/infrastructure/worksheet_deserializers_test.dart` (create if absent)

**Do NOT Modify**:
- `syncro-flutter/lib/features/ticket/ticket_details/worksheets/infrastructure/worksheet_repository_impl.dart`
- `syncro-flutter/lib/features/ticket/ticket_details/application/ticket_details_cubit.dart`
- Any other file

## Acceptance Criteria

- [ ] `fromJson({'worksheet_templates': [], 'metadata': {...}})` → returns empty list, no exception
- [ ] `fromJson({'worksheet_templates': [...]})` → returns worksheets (existing behavior unchanged)
- [ ] `fromJson({'totally_unknown_key': 'value'})` → still throws `FormatException` (unrecognized format)
- [ ] Test: empty `worksheet_templates` + `metadata` key → empty list returned
- [ ] Test: populated `worksheet_templates` → worksheets returned
- [ ] Test: unrecognized keys only → FormatException thrown
- [ ] `flutter test` passes
- [ ] `flutter analyze` passes
