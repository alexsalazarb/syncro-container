# Plan: SE-12477 — TimeClock Silent Push Sync

**Status**: complete
**Last Updated**: 2026-06-08

## Objective

Handle the `timelog_state_changed` silent FCM push sent by BE (SE-12420) to keep the TimeClock section and badge in sync when a timelog is created or updated from the web portal.

## Type

Type 1 — Product (user-facing behavior change)

## JIRA

[SE-12477](https://syncrotech.atlassian.net/browse/SE-12477)

## Master Plan

None

## Project

syncro-flutter

## Branch Convention

`plan/SE-12477-timeclock-push-sync/{task-path}`

## Demo Date

TBD

## Dev / QA

- **Dev**: Alex Salazar
- **QA**: Unassigned

---

## Context

The backend (SE-12420) sends a data-only FCM push to the `userID-{id}` topic whenever a timelog is
created or updated from the web portal. The most common trigger is a manager editing a **historical
timelog record** — not necessarily the active session.

The push payload:
```
{ "event": "timelog_state_changed", "timelog_id": "<id>", "state": "in" | "out" }
```

Mobile only uses `event` to detect the message. `timelog_id` and `state` are **not consumed**
because they describe the modified timelog, not the user's current clock state (e.g. a historical
edit sends `state: 'out'` even if the user has an active session). The push is a refresh trigger
only — current state is always sourced from the API.

---

## Architecture

```
FCM (data-only)
  │
  ├─ foreground → NotificationManager.onMessage listener
  │                   ↓ timelogStateChangedStream (broadcast)
  │                   ↓
  │               LatestTimeClockCubit._pushSubscription
  │                   ↓ refresh() → API call → safeEmit(LatestTimeClockLoaded)
  │                   ↓
  │               TimeClockButton (BlocBuilder) — green dot auto-updates
  │
  └─ background → _firebaseMessagingBackgroundHandler (top-level, isolate)
                      ↓ SharedPreferences.setBool('timelog_state_refresh_pending', true)
                      ↓
                  LatestTimeClockCubit._resumeSubscription
                      listens to AppLifecycleService.onResumedStream
                      ↓ checks + clears SP flag → refresh()
```

---

## Dependency DAG

```
task-01-fcm-handler  ──►  task-02-cubit-refresh-integration
```

task-02 depends on the `timelogStateChangedStream` added in task-01.

---

## Known Limitations

**Race condition — clock action submitted simultaneously with push arrival**

The push reduces the ghost-entry window to near zero, but does not eliminate it entirely.
If the user taps Clock Out on mobile in the same millisecond the push arrives and the refresh
is still in-flight, `isUpdating` blocks the push-refresh and the `PUT /timelogs` (no ID) is
sent to the server. If the manager already closed that session from the web, the server may
receive a redundant clock-out.

Full elimination requires BE to validate that an active session exists before processing
`PUT /timelogs` — i.e., an idempotent endpoint that returns an error (not a new entry) when
there is no open session. This is a BE responsibility outside the scope of this mobile ticket.

---

## Kill Criteria

1. `FirebaseMessaging.onBackgroundMessage` cannot be registered correctly in the isolate (platform restriction) — stop and escalate to team.
2. `LatestTimeClockCubit` is closed before the stream subscription fires (lifecycle issue) — redesign using a GetIt-registered service instead of cubit-level subscription.

---

## Tasks

| ID | Slug | Status | Owner Files |
|----|------|--------|-------------|
| task-01 | fcm-handler | complete | `notifications_manager.dart`, `main.dart` |
| task-02 | cubit-refresh-integration | complete | `latest_time_clock_cubit.dart`, new test file |

---

## Relevant KB

- `docs/kb-projects/syncro-flutter/technical/integrations/firebase.md` — FCM integration, NotificationManager pattern
- `docs/kb-projects/syncro-flutter/product/features/push-notifications.md` — push notification architecture
- `docs/kb-projects/syncro-flutter/technical/architecture/architecture-patterns.md` — Cubit patterns, safeEmit
