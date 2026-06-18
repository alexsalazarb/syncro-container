# task-06 — Chat: list + conversation

| Field | Value |
|-------|-------|
| **Phase** | 2 |
| **Branch** | `plan/patrol-deep-tests/phase-2/task-06-chat-deep` |
| **Routes covered** | `chats`, `chatDetail`, `cannedResponseCreateOrEdit` |
| **Depends on** | None |

---

## Objective

Replace the shallow `chat_test.dart` stub. Navigate into a chat conversation and verify the chat detail screen renders message input and history.

---

## File Ownership

**Modify:**
- `integration_test/patrol/features/chat_test.dart`

---

## Implementation Steps

1. In the authenticated path:
   a. Tap `'Chats'` tab, wait 3s (WebSocket connection may take time)
   b. Assert list renders (chat items or empty state)
   c. If items exist: tap first chat item → verify `ChatDetailPage` opens
   d. Assert chat detail: message input field visible (`find.byType(TextField)` or `find.text('Type a message...')`), message history list
   e. Do NOT send any messages
   f. Navigate back to chat list
   g. Look for "New Chat" button — verify it's tappable (don't open if it triggers backend creation)

## Key Widgets to Assert

| Screen | Find |
|--------|------|
| Chat list | `find.text('Chats')` + list items or empty state |
| Chat detail | Message input field, message list |
| Back | `find.text('Chats')` tab still visible |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/chat_test.dart \
  --flavor qa -d {device}
```

---

## Notes

- Chat uses a WebSocket (`phoenix_socket`) — expect a brief connection delay on open
- Do NOT tap send button or "New Chat" — both create backend data
- `cannedResponseCreateOrEdit` is accessed from the chat detail input area — verify it opens if a canned responses button exists, then dismiss immediately
