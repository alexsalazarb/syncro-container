# Task 07 — Chat send message + ticket timer entry add

| Field | Value |
|-------|-------|
| **Task ID** | task-07 |
| **Phase** | 2 — Mutations |
| **Status** | not-started |
| **Branch** | `plan/patrol-test-coverage-phase2/task-07-chat-timentry-mutations` |
| **JIRA** | N/A |

---

## Objective

Create `chat_timer_mutations_test.dart` covering two write-path flows:
1. **Chat send message** — type and send a message in a conversation
2. **Ticket timer entry add** — add a timer entry to a ticket (per-ticket, not global time clock)

---

## File Ownership

**Create:**
- `syncro-flutter/integration_test/patrol/features/chat_timer_mutations_test.dart`

**Do NOT modify:**
- `chat_test.dart`
- `time_clock_test.dart`

---

## Steps

### Chat send message
1. Navigate: Chats tab → first conversation → `chat_detail_page`
2. Find the message input `TextField`
3. Enter `[TEST] Integration test message`
4. Tap Send button (IconButton or similar)
5. Assert: message appears in the chat list OR input field is cleared (success indicator)
6. Navigate back to chat list

### Ticket timer entry add
1. Navigate: Tickets → first ticket → Details tab → find "Timer Entries" section
2. Tap to open `timer_entries_page`
3. Tap add FAB → `add_edit_timer_entry_page`
4. Fill required fields:
   - Start time (current time or a text field)
   - Duration or end time if required
   - Note/description: `[TEST] Integration test timer entry`
5. Tap Save
6. Assert: redirected to `timer_entries_page` OR success indicator visible
7. Navigate back to tickets list

### Caveats
- Chat messages sent to a real technician conversation — use a test/demo conversation if available in QA
- Timer entries mutate ticket data — use a low-priority test ticket if possible
- Both wrapped in `withSocketFilter`

---

## Testing

Run: `fvm flutter test integration_test/patrol/features/chat_timer_mutations_test.dart --flavor qa -d {device}`

Expected: both mutations succeed and app navigates to expected post-save state.
