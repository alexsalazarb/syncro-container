# Asset Domain Concept

**Last Updated**: April 2026

## Context

An Asset is a managed device or piece of equipment belonging to a customer. Assets are the primary subject of RMM (Remote Monitoring & Management) operations.

## Core Properties

- `id` — numeric asset identifier
- `name` — device name (e.g., "DESKTOP-ABC123")
- `customer` — owning customer company
- `type` — device category (workstation, server, printer, etc.)
- RMM status, OS info, last seen timestamp

## Key Operations

- **View** — see asset details, hardware info, current status
- **Run scripts** — execute remote scripts (requires session/2FA token)
- **Add to queue** — schedule a script to run
- **Chat** — start a real-time chat conversation linked to this asset
- **View alerts** — see RMM monitoring alerts for the asset

## Session Token for Scripts

RMM script execution requires a secondary authorization token (`sessionToken`). This is injected into the request header as `Authorization2FAToken`. The token is obtained separately from the main auth flow.

## Relationship to Chat

Every chat interaction is asset-based. The Phoenix socket channel topic for a chat is `account:asset:{accountId}:{assetId}`. A chat cannot exist without an associated asset.

## Flutter Feature

- `assets` feature — list, filter, search, barcode scan
- `asset_detail` feature — detail view, scripts, add-to-queue
- `alerts` feature — RMM alerts (may replace assets tab depending on feature flag)
