# Task: Appointments & Misc Domain Models — fromMap → fromJson

**Plan**: Standardize Domain Model Serialization (fromJson / toJson)
**Phase**: 2
**Task ID**: task-04
**Task Path**: phase-2/task-04-appointments-misc-models
**Depends On**: phase-1/task-01-core-shared-models
**JIRA**: N/A

## Objective

Rename `fromMap` / `toMap` / `fromJson(String)` in appointments, alerts, and time-clock domain models; update corresponding tests and infrastructure deserializers.

## Context

`AppointmentPageArgs` and `TicketData` each have **both** `fromMap(Map)` and `fromJson(String source)` — the String overload must be deleted and its callers updated to use `jsonDecode()` inline.

`Alert.toMap()` exists in `alerts.dart` — rename to `toJson()`.

`GetAppointmentByIdResponse` and `GetTimeClockListResponse` / `TimeClockResponse` have `fromMap` definitions — rename to `fromJson`.

`Appointment.fromJson` and `appointment.dart` call sites must be verified — `Appointment` itself already uses `fromJson` but may internally call other `fromMap` methods (check before starting).

This task can run in parallel with task-02 and task-03.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Confirm task-01 is **complete** (check `phase-1/task-01-core-shared-models/status.md`)
- [ ] Read `syncro-flutter/lib/features/appointments/appointment_create/domain/appointment_page_args.dart` — has `fromMap` + `fromJson(String)` + `TicketData` inner class with the same pattern
- [ ] Read `syncro-flutter/lib/features/appointments/core/domain/appointment.dart` — verify it only has call sites, not definitions to rename
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `syncro-flutter/lib/features/appointments/appointment_create/domain/appointment_page_args.dart` | modify | `AppointmentPageArgs.fromMap` → `fromJson`; remove `fromJson(String)`; same for `TicketData`; rename `toMap` → `toJson` |
| `syncro-flutter/lib/features/appointments/core/domain/get_appointment_by_id_response.dart` | modify | `fromMap` → `fromJson` |
| `syncro-flutter/lib/features/appointments/core/domain/appointment.dart` | modify | Update any `.fromMap(` call sites if present (the class itself likely already uses `fromJson`) |
| `syncro-flutter/lib/features/alerts/domain/alerts.dart` | modify | `Alert.toMap()` → `Alert.toJson()`; check for `fromMap` definition and rename if found |
| `syncro-flutter/lib/features/time_clock/domain/time_clock_response.dart` | modify | `toMap` → `toJson`; verify `fromMap` and rename if found |
| `syncro-flutter/lib/features/time_clock/domain/get_time_clock_list_response.dart` | modify | `toMap` → `toJson`; verify `fromMap` and rename if found |
| `syncro-flutter/lib/features/appointments/core/infrastructure/get_appointment_by_id_deserializer.dart` | modify | Update `.fromMap(` call sites |
| `syncro-flutter/test/features/appointments/appointment_create/domain/appointment_page_args_test.dart` | modify | `fromMap` → `fromJson`, `fromJson(String)` → `fromJson(jsonDecode(...))`, `toMap` → `toJson` |

### Do NOT Modify

- Any file under `features/customers/` — owned by task-01
- Any file under `features/ticket/` — owned by task-02 and task-03
- Any remaining infrastructure files not listed above — owned by task-05

## Implementation Steps

### Step 1: Migrate `appointment_page_args.dart`

1. `AppointmentPageArgs.fromMap(Map)` → `fromJson(Map)` (rename param to `json`)
2. `AppointmentPageArgs.fromJson(String source)` → **delete**; grep for callers and replace with `AppointmentPageArgs.fromJson(jsonDecode(source) as Map<String, dynamic>)`
3. `AppointmentPageArgs.toMap()` → `toJson()` — grep for callers and update
4. Apply the same 3-step pattern to `TicketData` inside the same file

### Step 2: Rename in `get_appointment_by_id_response.dart`

1. `GetAppointmentByIdResponse.fromMap(...)` → `fromJson(...)`

### Step 3: Verify `appointment.dart` call sites

1. `grep -n "fromMap\|fromJson(String"` in the file
2. If any `fromMap` or `String`-overload call sites exist, update them; if not, no changes needed

### Step 4: Rename `toMap` in `alerts.dart`, `time_clock_response.dart`, `get_time_clock_list_response.dart`

For each:
1. Rename `toMap()` → `toJson()`
2. Grep for callers of `.toMap()` on these types and update them
3. Check for `fromMap` definition — rename to `fromJson` if found

### Step 5: Update infrastructure and test call sites

For each infrastructure / test file listed in File Ownership:
- Replace `.fromMap(x)` → `.fromJson(x)` for affected types
- Replace `X.fromJson(rawStr)` → `X.fromJson(jsonDecode(rawStr) as Map<String, dynamic>)` where applicable
- Replace `.toMap()` → `.toJson()` where applicable

### Step 6: Verify

```bash
cd syncro-flutter
flutter analyze
flutter test test/features/appointments/ test/features/alerts/ test/features/time_clock/
```

## Testing

- [ ] All tests in `test/features/appointments/` pass
- [ ] `flutter analyze` passes with no new issues
- [ ] `grep -rn "fromMap\|\.toMap()" syncro-flutter/lib/features/appointments/ syncro-flutter/lib/features/alerts/ syncro-flutter/lib/features/time_clock/` returns no results
- [ ] No `fromJson(String` overload remains in any owned domain file

## Documentation / KB Updates

- [ ] No new KB doc needed

## Completion Criteria

- [ ] All `fromMap` factories renamed in owned domain files
- [ ] All `toMap` methods renamed in owned domain files
- [ ] All `fromJson(String)` overloads removed; call sites updated to use `jsonDecode()`
- [ ] Tests pass for all covered areas
- [ ] `flutter analyze` passes with no new issues
- [ ] Changes committed to `plan/standardize-serialization/phase-2/task-04-appointments-misc-models` branch
- [ ] Status updated in `status.md`
