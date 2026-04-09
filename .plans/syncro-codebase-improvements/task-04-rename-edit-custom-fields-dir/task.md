# Task 04 — Rename edit-custom-fields Directory

**Plan**: syncro-codebase-improvements
**Priority**: 🟡 Minor
**Branch**: `plan/syncro-codebase-improvements/task-04`

## Problem

A directory has a space in its name:

```
lib/features/ticket/ticket_custom_fields/edit custom fields/
```

This causes:
- URL-encoded imports (`edit%20custom%20fields`) in `route_cubit.dart`
- Potential issues on case-sensitive filesystems and CI tooling
- Shell script failures if not properly quoted

## Objective

Rename to `edit_custom_fields/` (snake_case, no space).

## Implementation Steps

1. **Rename directory**:
   ```bash
   cd lib/features/ticket/ticket_custom_fields/
   mv "edit custom fields" edit_custom_fields
   ```

2. **Update all imports** referencing the old path:
   - Search for `edit%20custom%20fields` and `edit custom fields` in all `.dart` files
   - Update to `edit_custom_fields`

3. **Update `route_cubit.dart`** import that uses the URL-encoded path

4. **Run `flutter analyze`** to catch broken imports

5. **Run `flutter test`**

## Files to Modify

- Rename: `lib/features/ticket/ticket_custom_fields/edit custom fields/` → `edit_custom_fields/`
- All files importing from that path

## Acceptance Criteria

- [ ] Directory renamed with no spaces
- [ ] All imports updated
- [ ] `flutter analyze` passes with no errors
- [ ] `flutter test` passes
