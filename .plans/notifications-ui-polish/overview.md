# Plan: Notifications UI Polish

**Status**: not-started
**Created**: 2026-05-12
**Last Updated**: 2026-05-12
**Type**: Feature continuation
**Ticket**: SE-11982 / SE-11984 (continuation — no new tickets)
**Branch**: feature/notifications
**Master Plan**: None

## Context

Continuation of the `notifications-screen` plan (all 5 tasks complete). This plan addresses UI/UX polish discovered during QA:

1. The unread badge on the dashboard does not update after marking notifications as read inside the Notifications screen.
2. The filter toggle (TextButton) must be replaced with a BottomSheet picker (design spec).
3. The AppBar must show "Mark all Read" as an outlined button + a filter icon (design spec).
4. The notification item layout needs: 3-line body, time ago + chevron right at top-right, and swipe on unread items to "Mark as Read".

## Design Reference

- Filter BottomSheet: modal with "Select Filter" title, checkmark on active option ("Hide Read" / "View All"), "Cancel" button.
- AppBar: back arrow | "Notifications" title | "Mark all Read" (outlined) | filter icon.
- Swipe on unread item: reveals gray background with ✕ icon + "Mark as Read" label on the right.

## Scope

**Project**: syncro-flutter
**Branch**: `feature/notifications` (no new branch — continues existing work)

## Tasks

| Task Path | Title | Status | Depends On |
|-----------|-------|--------|------------|
| task-01-badge-refresh | Sync unread badge on return to dashboard | not-started | — |
| task-02-appbar-filter-bottomsheet | AppBar redesign + filter BottomSheet | not-started | — |
| task-03-item-widget-redesign | Item widget layout: 3 lines, time+chevron, swipe to mark-as-read | not-started | — |

All three tasks are independent — no cross-dependencies.

## Key Files

| File | Relevance |
|------|-----------|
| `lib/features/notifications/presentation/notifications_page.dart` | AppBar + page shell + filter logic |
| `lib/features/notifications/presentation/widgets/notification_item_widget.dart` | Item layout + swipe |
| `lib/features/notifications/application/unread_count_cubit.dart` | Badge count — `fetchUnreadCount()` |
| `lib/core/design_system/app_strings.dart` | String constants |

## Success Criteria

- [ ] Unread badge on dashboard reflects correct count after returning from Notifications screen
- [ ] Filter opens as a BottomSheet with checkmark on active option
- [ ] AppBar shows "Mark all Read" outlined button + filter icon
- [ ] Notification body shows up to 3 lines
- [ ] Time ago and chevron right appear together at top-right of each item
- [ ] Swiping an unread notification reveals "Mark as Read" action
- [ ] All existing tests pass
