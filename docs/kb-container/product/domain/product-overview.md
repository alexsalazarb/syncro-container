# Product Overview — Syncro MSP

**Last Updated**: April 2026

## Context

Syncro is a **Managed Service Provider (MSP) platform** for IT service companies. The mobile app is used by field technicians to manage their daily service work from their phones.

## What the Product Does

Syncro enables MSP technicians to:
- View and manage **service tickets** (create, update, add notes, track time, add charges)
- Monitor **customer assets** (managed devices) with RMM alerts and remote scripts
- **Chat** in real-time with end-users/customers about their devices
- **Schedule and manage appointments**
- Track **work time** with a time clock
- Access **customer and contact** information

## Platform Matrix

| Platform | Stack | Status |
|----------|-------|--------|
| iOS + Android | Flutter (this repo) | Production |

## Glossary

| Term | Definition |
|------|-----------|
| MSP | Managed Service Provider — IT company that manages customers' devices |
| Technician | A user of the Syncro mobile app; an IT professional servicing customer assets |
| Ticket | A service request record — the primary unit of work in MSP workflows |
| Asset | A managed device (computer, server, printer, etc.) belonging to a customer |
| Interaction / Chat | A real-time text conversation between a technician and a customer about an asset |
| Account | The MSP company's Syncro account — all data is scoped per account |
| Customer | A business entity (company) that the MSP services |
| End User / Contact | An individual person at a customer company |
| RMM | Remote Monitoring & Management — automated monitoring and control of assets |
| Alert | An automated RMM notification that an asset needs attention |
| Worksheet | A digital form/checklist attached to a ticket |
| Timer Entry | A logged block of time spent working on a ticket (used for billing) |
| Charge | A billable line item on a ticket (labor, parts, services) |
| Canned Response | A pre-written note template for common ticket updates |
| Session Token | Secondary auth token required for RMM script execution |

## Multi-Environment

The app supports multiple backend environments selected at runtime (if feature flag is enabled):
- `qa` — QA environment
- `ss1`–`ss12` — Staging servers 1–12
- `prod` — Production

In production builds, `showSwitchEnvironment` is `false` so users always connect to `prod`.
