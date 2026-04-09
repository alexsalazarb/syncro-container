# Ticket Management Feature — syncro-flutter

**Last Updated**: April 2026

## Context

Tickets are the core workflow of the Syncro app. They represent service requests that technicians work on. The ticket feature is the most complex in the app with 15+ routes and multiple sub-features.

## Sub-Features

| Sub-feature | Path | Description |
|------------|------|-------------|
| Ticket list | `ticket/ticket_home/` | Paginated list with filter/search |
| Ticket create | `ticket/ticket_create/` | Create new ticket from scratch or from alert |
| Ticket details | `ticket/ticket_details/` | Full ticket view with tabs |
| Notes | `ticket/ticket_details/create_note/` | Add text notes to a ticket |
| Canned responses | `create_note/canned_responses/` | Pre-written note templates |
| Worksheets | `ticket/ticket_details/worksheets/` | Digital forms attached to tickets |
| Timer entries | `ticket/timer_entries/` | Time tracking per ticket |
| Ticket charges | `ticket/ticket_charges/` | Billable charges on tickets |
| Custom fields | `ticket/ticket_custom_fields/` | Custom metadata fields |
| Attachments | `ticket/ticket_attachment/` | File attachments and preview |
| Search | `ticket/ticket_search/` | Full-text ticket search |
| Shared settings | `ticket/shared/` | `TicketsSettingsCubit` — statuses, priorities, technicians |

## Core Cubits

- `TicketCubit` — ticket list, pagination, search
- `TicketFilterCubit` — filter state (depends on `TicketsSettingsCubit`)
- `TicketsSettingsCubit` — loads ticket statuses/priorities/technicians from API; shared by filter and create
- `TicketDetailsCubit` — single ticket state; passed as route parameter to sub-screens (timer entries, charges, worksheets, notes)

## ⚠️ TicketDetailsCubit as Route Parameter

`TicketDetailsCubit` is passed through `GoRouter`'s `state.extra` to sub-routes. This means the same cubit instance is shared between the detail page and all its child pages:

```dart
_createTypedRoute<TicketDetailsCubit>(
  route: AppRoute.ticketNoteCreate,
  childBuilder: (ticketDetailsCubit) => CreateNotePage(
    ticketId: ticketDetailsCubit.ticketId,
    ticketDetailsCubit: ticketDetailsCubit,
  ),
)
```

This allows child pages to emit state back to the parent detail page, but it couples navigation to a specific cubit instance. Do not dispose the cubit in child pages.

## Worksheets

Worksheets are digital forms (templates + responses) attached to tickets. They have their own CRUD sub-flow:
- `WorksheetCreatePage` — select template + create
- `WorksheetDetailEditPage` — fill out worksheet fields
- `WorksheetPageHistory` — view change history

`WorksheetRepository` is registered in GetIt — prefer `getIt<WorksheetRepository>()` unless calling from a context that has the widget-tree registration.

## Ticket Settings (Shared State)

`TicketsSettingsCubit` is the most widely-used shared cubit. It loads:
- Ticket statuses (open, closed, etc.)
- Priority levels
- Technician list
- Contact list (from `EndUsersRepository`)
- Customer list

It is registered in `AppProviders` and **must be registered before** `TicketFilterCubit` and `TechniciansCubit` since they depend on it.
