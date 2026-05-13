# Status: Item widget layout — 3 lines, time+chevron, swipe to mark-as-read

**Current Status**: complete
**Last Updated**: 2026-05-12
**Agent**: claude-sonnet-4-6
**Branch**: feature/notifications
**PR**: —

## Status History

| Timestamp | Status | Agent | Notes |
|-----------|--------|-------|-------|
| 2026-05-12 | not-started | — | Task created |
| 2026-05-12 | complete | claude-sonnet-4-6 | Item widget redesigned: 3 lines, time+chevron, swipe-to-mark-read |

## Blockers

None

## Artifacts

- `syncro-flutter/lib/features/notifications/presentation/widgets/notification_item_widget.dart` — full redesign: InkWell+Column layout, 3-line body, time+chevron top-right, Dismissible on unread items only
- `syncro-flutter/lib/features/notifications/presentation/notifications_page.dart` — updated `onMarkAsUnread` → `onMarkAsRead` callback
- `syncro-flutter/lib/core/design_system/app_strings.dart` — added `notificationsMarkAsRead` constant
- `syncro-flutter/test/features/notifications/presentation/notification_item_widget_test.dart` — updated all tests: `onMarkAsUnread` → `onMarkAsRead`, Dismissible logic inverted, ListTile → InkWell

## Adaptations

- Used `AppStrings.notificationsMarkAsRead` instead of inline string (constant did not exist, added it)
- Removed unused `theme_extension.dart` import (no longer needed since `context.primaryMain` / `context.backgroundDefault` are not used in the new design)
