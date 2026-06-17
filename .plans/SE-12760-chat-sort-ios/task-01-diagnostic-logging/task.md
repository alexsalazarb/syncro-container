# Task: Inspect interactions_batched Payload via Charles Proxy

**Plan**: SE-12760-chat-sort-ios
**Task ID**: task-01
**Task Path**: task-01-diagnostic-logging
**Depends On**: None
**Ticket**: SE-12760

## Objective

Use Charles Proxy to intercept the `interactions_batched` WebSocket payload on your own iOS device. The goal is NOT to reproduce the sort bug (your account always shows correctly) — it's to understand the full payload structure: what timestamp fields are available, what `inserted_at` looks like, and whether there are other fields the web app could be using to sort differently.

This eliminates the blindness before shipping task-02/03.

## Context

We know `_sortChats()` is correct and always called. The working hypothesis is that `last_message.inserted_at` from the socket doesn't match the "last activity" ordering the web app uses — but we can't confirm this without seeing actual payload data. Charles Proxy on your device gives us that data without needing the customer's device.

See [investigation.md](../investigation.md) Finding 6.

## Before You Start

- [ ] Charles Proxy installed and licensed (or use mitmproxy as alternative)
- [ ] iPhone configured to use Charles as HTTP proxy
- [ ] Charles SSL certificate installed on device (required for WSS interception)
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## Setup Steps

### Step 1: Configure Charles Proxy for WebSocket

1. Open Charles → **Proxy → Proxy Settings** → Port `8888`
2. On iPhone: Settings → Wi-Fi → your network → HTTP Proxy → Manual → host: Mac's IP, port: 8888
3. Install Charles root certificate on device: Charles → **Help → SSL Proxying → Install Charles Root Certificate on Mobile Device**
4. On iPhone: Settings → General → VPN & Device Management → trust the certificate
5. In Charles: **Proxy → SSL Proxying Settings** → add `*.syncromsp.com` (or `*` for all)

### Step 2: Capture the interactions_batched event

1. Open the Syncro app on iPhone and navigate to the Chats tab
2. In Charles, look for WebSocket connections — filter by `syncromsp.com`
3. Find the WebSocket upgrade handshake, then look for frames containing `interactions_batched`
4. Copy the full JSON payload of the `interactions_batched` response

**Alternative**: Use Charles' **"Save Session"** and look through saved frames if the WebSocket connection is fast.

### Step 3: Analyze the payload

For EACH chat entry in the payload, document:

| Field | Example value | Notes |
|-------|---------------|-------|
| `id` | `123` | |
| `last_message.inserted_at` | `"2026-06-15T10:00:00"` | What we sort by |
| `last_message.created_at` | present? | Does this field exist? |
| `last_message.updated_at` | present? | Does this field exist? |
| `updated_at` (top-level) | present? | Does the interaction have its own timestamp? |
| Any other timestamp fields | — | List them |

Compare the `inserted_at` values against the order shown in the web app at your own account's chat URL. If they match → sort is correct for your data. If not → sort is wrong even for your account (you might not notice because your chats change frequently).

### Step 4: Compare web app sort field

Open your account's web chat list in a browser with DevTools open. Check the Network tab for the REST call to `/chat/all` or similar. Look at what field the server returns and how the response is ordered. Is it the same `inserted_at`, or a different field?

### Step 5: Document findings in status.md

Record:
- Full list of timestamp fields available in the payload
- Whether `inserted_at` order matches web app order for your account
- Whether there are other timestamp fields that could be used for sorting
- Decision: **mobile-only fix possible** (use a different field) OR **backend issue** (server sends wrong values → file backend ticket)

## Testing

- [ ] `fvm flutter analyze` — no source changes needed for this task
- [ ] Document findings before marking complete

## Documentation / KB Updates

- [ ] No code changes in this task — findings documented in `status.md` only
- [ ] If a backend issue is confirmed, file a new Jira ticket against the backend team and link it here

## Completion Criteria

- [ ] `interactions_batched` payload captured and analyzed
- [ ] All timestamp fields documented in `status.md`
- [ ] Comparison against web app sort order completed
- [ ] Decision made: mobile fix (adapt task-02/03) or backend ticket needed
- [ ] If a new timestamp field is available for sorting → update task-03 to use it instead of `inserted_at`
- [ ] Status updated in `status.md`
