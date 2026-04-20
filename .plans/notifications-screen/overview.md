# Plan: Notifications Screen

**Status**: not-started
**Created**: 2026-04-16
**Last Updated**: 2026-04-16
**Estimated Demo Date**: TBD
**Assigned Dev**: unassigned
**Assigned QA**: unassigned
**Master Plan**: None
**Integration Branch**: plan/notifications-screen — populated by `execute-plan` at start
**Base Branch**: main

## Objective

Build the Notifications Screen for Syncro Mobile, allowing users to view, filter, and manage in-app notifications (read/unread state), with deep-link navigation to related objects and an unread badge on the home screen.

## Scope

### In Scope
- `NotificationItem` domain model matching the BE payload
- Repository + use cases: fetch list, get unread count, mark as read, mark as unread, mark all as read
- `NotificationsCubit` with pagination, filter toggle (unread / all), and optimistic updates
- Notifications screen UI: paginated list, pull-to-refresh, swipe-to-unread, empty/error states
- Home screen unread badge (bell icon with counter)
- Tap → navigate via existing `redirectOnNotification()` + auto-mark-as-read
- All 5 `object_type` routing paths validated: ticket, appointment, comment, rmmalert, openstruct

### Out of Scope
- Push notification delivery infrastructure — handled separately
- Backend endpoints (SE-11976, SE-11977) — owned by BE team
- Changes to `PushNotification` model or `redirectOnNotification()` core logic — reused as-is
- Notification preferences / settings screen

## Kill Criteria

- BE endpoints (SE-11976, SE-11977) not delivered in time for mobile integration
- `object_type` values change from the agreed strings (ticket, appointment, comment, rmmalert, openstruct)
- Routing for any of the 5 object types breaks due to upstream changes

## Phases

| Phase | Name | Tasks | Dependencies | Description |
|-------|------|-------|--------------|-------------|
| 1 | Foundation | task-01 | None | Domain model, repository, use cases |
| 2 | Logic | task-02, task-03 | Phase 1 | Cubit + state, home screen badge |
| 3 | UI | task-04 | Phase 2 (task-02) | Notifications screen, pagination, swipe |
| 4 | Routing | task-05 | Phase 2 (task-02), Phase 3 | Tap → deep-link + auto-mark-read |

## Task Summary

| Task Path | Title | Phase | JIRA | Status | Depends On |
|-----------|-------|-------|------|--------|------------|
| phase-1/task-01-data-layer | Notifications data layer | 1 | SE-11980 | not-started | — |
| phase-2/task-02-notifications-cubit | NotificationsCubit + state | 2 | SE-11981 | not-started | phase-1/task-01-data-layer |
| phase-2/task-03-home-badge | Home screen unread badge | 2 | SE-11983 | not-started | phase-1/task-01-data-layer |
| phase-3/task-04-notifications-screen | Notifications screen UI | 3 | SE-11982 | not-started | phase-2/task-02-notifications-cubit |
| phase-4/task-05-tap-routing | onTap routing + auto-mark-read | 4 | SE-11984 | not-started | phase-2/task-02-notifications-cubit, phase-3/task-04-notifications-screen |

## Branch Convention

Pattern: `plan/notifications-screen/{task-path}`

Examples:
- `plan/notifications-screen/phase-1/task-01-data-layer`
- `plan/notifications-screen/phase-2/task-02-notifications-cubit`

Base branch: main

## Key Files

| File/Directory | Relevance |
|----------------|-----------|
| `syncro-flutter/lib/features/notifications/` | New feature directory (all layers) |
| `syncro-flutter/lib/core/routing/route_cubit.dart` | `redirectOnNotification()` — reused for routing |
| `syncro-flutter/lib/core/services/push_notification/push_notification.dart` | `PushNotification` model — reused for tap routing |
| `syncro-flutter/lib/core/routing/` | GoRouter navigation |
| `syncro-flutter/test/features/notifications/` | Test directory for all tasks |

## BE Dependencies

| Endpoint | JIRA | Purpose |
|----------|------|---------|
| `GET /notifications` | SE-11976 | Paginated list with filter |
| `GET /notifications/unread_count` | SE-11976 | Badge counter |
| `PATCH /notifications/:id/read` | SE-11977 | Mark single as read |
| `PATCH /notifications/:id/unread` | SE-11977 | Mark single as unread |
| `PATCH /notifications/mark_all_read` | SE-11977 | Mark all as read |

## Payload Contract

```json
{
  "id": 123,
  "title": "string",
  "description": "string",
  "is_read": false,
  "created_at": "2026-04-16T12:00:00Z",
  "object_type": "ticket",
  "object_id": 456,
  "object_link": "https://..."
}
```

`object_type` values (must match existing `NotificationSource` enum strings exactly):
`ticket` | `appointment` | `comment` | `rmmalert` | `openstruct`

## Risks

- BE endpoint delivery delayed — Mitigation: stub repository with mock data during Phase 1-2 development
- `object_type` string mismatch — Mitigation: validate against `_parseNotificationSource()` in route_cubit.dart before integration
- Singleton service vs RepositoryProvider pattern conflict — Mitigation: follow project convention (RepositoryProvider in widget tree, NOT GetIt)

## Defects

| Defect Task | Title | Found During | Blocks | Status |
|-------------|-------|-------------|--------|--------|

## Success Criteria

- [ ] `NotificationItem` domain model deserializes correctly from BE payload
- [ ] All 5 use cases implemented with `Either<Failure, T>` pattern
- [ ] `NotificationsCubit` handles pagination, filter toggle, optimistic read/unread toggle
- [ ] Notifications screen shows paginated list with pull-to-refresh, swipe gesture, empty/error states
- [ ] Home screen badge shows unread count, refreshes on app foreground
- [ ] Tapping a notification navigates to correct screen for all 5 `object_type` values
- [ ] Auto-mark-as-read fires on tap
- [ ] All tests pass (`flutter test`)
- [ ] `flutter analyze` passes

## References

- **JIRA Spike**: [SE-11964](https://repairtechsolutions.atlassian.net/browse/SE-11964)
- **BE Read Endpoints**: [SE-11976](https://repairtechsolutions.atlassian.net/browse/SE-11976)
- **BE Write Endpoints**: [SE-11977](https://repairtechsolutions.atlassian.net/browse/SE-11977)
- **Mobile Tickets**: SE-11980, SE-11981, SE-11982, SE-11983, SE-11984
- **Related Plans**: None
