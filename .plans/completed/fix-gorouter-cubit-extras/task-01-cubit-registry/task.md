# Task: Create TicketCubitRegistry + self-register in TicketDetailsCubit

**Plan**: Fix GoRouter Cubit Extras Serialization Warning
**Task ID**: task-01
**Task Path**: task-01-cubit-registry
**Depends On**: None
**JIRA**: N/A

## Objective

Create `TicketCubitRegistry`, a lightweight in-memory singleton that maps `ticketId → TicketDetailsCubit`. Wire lifecycle registration directly into `TicketDetailsCubit` so it self-registers on construction and self-unregisters when closed. This registry is the foundation all subsequent tasks depend on.

## Context

GoRouter v14 emits a serialization warning whenever a non-primitive (e.g. a Cubit) is passed as route `extra`. The root cause is that `TicketDetailsCubit` is passed directly to 6 sub-routes so child pages can share the same live instance. The fix is to decouple the cubit from the router by storing it in a registry keyed by ticket ID, then passing only the primitive `ticketId` as extra.

**Why self-registration in the cubit?**
- `TicketDetailsCubit` is created inside `BlocProvider.create` in `TicketDetailsPage`, giving us a hook at construction time
- `close()` is always called by `BlocProvider` when the page leaves the widget tree, giving a reliable cleanup hook
- No `StatefulWidget` conversion needed — less diff, fewer moving parts

**Relevant KB**: `docs/kb-projects/syncro-flutter/README.md` → architecture-patterns, coding-conventions

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Check `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read `lib/features/ticket/ticket_details/application/ticket_details_cubit.dart` fully
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/ticket/shared/ticket_cubit_registry.dart` | create | New registry singleton |
| `lib/features/ticket/ticket_details/application/ticket_details_cubit.dart` | modify | Add `TicketCubitRegistry.instance.register()` in constructor + `unregister()` in `close()` |

### Do NOT Modify

- `lib/core/routing/app_router.dart` — owned by task-02-router-and-params
- Any presentation or view files — owned by task-03 and task-04

## Implementation Steps

### Step 1: Create `TicketCubitRegistry`

Create `lib/features/ticket/shared/ticket_cubit_registry.dart`:

```dart
import 'package:syncro/features/ticket/ticket_details/application/ticket_details_cubit.dart';

/// In-memory registry that maps a ticket ID to its live [TicketDetailsCubit].
///
/// Child pages navigated to from [TicketDetailsPage] retrieve the shared cubit
/// here instead of receiving it through GoRouter extras (which cannot be
/// serialized and trigger GoRouter warnings).
///
/// Lifecycle is fully managed by [TicketDetailsCubit] itself:
///   - registers on construction
///   - unregisters in [TicketDetailsCubit.close]
class TicketCubitRegistry {
  TicketCubitRegistry._();

  static final TicketCubitRegistry instance = TicketCubitRegistry._();

  final Map<int, TicketDetailsCubit> _cubits = {};

  /// Registers [cubit] under [ticketId]. Overwrites any stale entry for the
  /// same ticket (e.g. after the app re-opens the same ticket).
  void register(int ticketId, TicketDetailsCubit cubit) {
    _cubits[ticketId] = cubit;
  }

  /// Returns the live [TicketDetailsCubit] for [ticketId], or `null` if not
  /// registered (should not happen in normal navigation flow).
  TicketDetailsCubit? get(int ticketId) => _cubits[ticketId];

  /// Removes the entry for [ticketId]. Called from [TicketDetailsCubit.close].
  void unregister(int ticketId) {
    _cubits.remove(ticketId);
  }
}
```

### Step 2: Wire into `TicketDetailsCubit`

In `ticket_details_cubit.dart`:

1. Import `TicketCubitRegistry`.
2. At the **end of the constructor body**, call:
   ```dart
   TicketCubitRegistry.instance.register(ticketId, this);
   ```
3. Override `close()`:
   ```dart
   @override
   Future<void> close() {
     TicketCubitRegistry.instance.unregister(ticketId);
     return super.close();
   }
   ```

> `ticketId` is the non-final `int ticketId` field already on the cubit. It may be updated by `getTicketByNumber` — ensure you read the current value at unregister time (which is fine since unregister is called at close, after the final ticketId is set).

## Testing

- [ ] Unit test `TicketCubitRegistry`:
  - `register` + `get` returns the registered cubit
  - `unregister` makes `get` return null
  - Overwrite: re-registering same ticketId returns the new cubit
- [ ] Verify `TicketDetailsCubit` unit tests still pass after the constructor/close changes
- [ ] `flutter analyze` passes — no new lint warnings
- [ ] `pre-commit-check` passes

## Documentation / KB Updates

- [ ] No KB/doc updates required — this is an implementation detail

## Completion Criteria

- [ ] `TicketCubitRegistry` singleton created and tested
- [ ] `TicketDetailsCubit` registers itself on construction and unregisters on close
- [ ] All existing `TicketDetailsCubit` tests pass
- [ ] `flutter analyze` clean
- [ ] Status updated in `status.md`
- [ ] Changes committed to `plan/fix-gorouter-cubit-extras/task-01-cubit-registry` branch
