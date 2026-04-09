# Ticket Domain Concept

**Last Updated**: April 2026

## Context

The Ticket is the primary unit of work in the Syncro MSP platform. It represents a service request that a technician must resolve.

## Core Properties

- `id` — numeric ticket identifier
- `number` — human-readable ticket number (shown in UI)
- `status` — current lifecycle state (Open, In Progress, Resolved, Closed, etc.)
- `priority` — urgency level (Low, Medium, High, Critical)
- `subject` — brief description of the issue
- `assignee` — technician assigned to the ticket
- `customer` — customer company the ticket belongs to
- `contact` — specific end-user/contact who reported the issue
- `asset` — optionally linked managed device

## Ticket Lifecycle

```
Created → In Progress → Resolved → Closed
              ↑               |
              └───────────────┘ (can reopen)
```

Statuses and priorities are loaded via `TicketsSettingsCubit` from the API. They are not hardcoded — different accounts may have different status names.

## Sub-Objects

| Sub-object | Description |
|-----------|-------------|
| Note | Text update added to the ticket timeline |
| Timer Entry | Time block (start/end) logged against the ticket for billing |
| Charge | Billable line item (labor, parts, services) |
| Worksheet | Digital form/checklist attached to the ticket |
| Attachment | File uploaded to the ticket |
| Custom Field | Account-specific metadata fields |

## Key Relationships

- A ticket belongs to a `Customer`
- A ticket may reference an `Asset` (managed device)
- A ticket may have a `Contact` (end user who filed it)
- A ticket has many `Notes`, `TimerEntries`, `Charges`, `Worksheets`, `Attachments`
- A ticket can be created from a `Chat` interaction or an `Alert`

## Flutter Feature

The `ticket` feature module handles all ticket operations. See `docs/kb-projects/syncro-flutter/product/features/ticket-management.md`.
