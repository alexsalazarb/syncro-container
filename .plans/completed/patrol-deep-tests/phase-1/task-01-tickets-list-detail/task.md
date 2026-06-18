# task-01 — Tickets: list + detail navigation

| Field | Value |
|-------|-------|
| **Phase** | 1 |
| **Branch** | `plan/patrol-deep-tests/phase-1/task-01-tickets-list-detail` |
| **Routes covered** | `tickets`, `ticketDetail` |
| **Depends on** | None |

---

## Objective

Replace the shallow `tickets_test.dart` stub with a test that navigates from the Tickets tab into a real ticket detail screen and verifies key widgets on both screens.

---

## File Ownership

**Modify:**
- `integration_test/patrol/features/tickets_test.dart`
- `integration_test/patrol/utils/test_keys.dart` (add keys if needed)

**Do NOT modify:**
- Any other `integration_test/patrol/features/*.dart`
- `integration_test/patrol/utils/app_test_setup.dart`
- Any `lib/` source files (add widget Keys only if strictly necessary)

---

## Implementation Steps

1. Open `integration_test/patrol/features/tickets_test.dart`
2. Keep the existing `patrolWidgetTest` structure (one test per file rule)
3. In the authenticated path:
   a. Tap the `'Tickets'` nav tab and wait 3s for list to load
   b. Verify the Tickets list screen is showing: look for a `ListView` or `find.byType(RefreshIndicator)`
   c. If list is empty: assert an empty-state widget exists and stop — do NOT fail
   d. If list has items: tap the first visible item (`find.byType(ListTile).first` or `find.text` on a known widget)
   e. Wait 2s for detail screen to load
   f. Assert detail screen: look for tab bar (Notes / Charges / Fields / Attachments tabs) or a ticket subject text
   g. Navigate back with `$.pageBack()` or `find.byTooltip('Back')`
   h. Assert we're back on the list (Tickets tab visible)
4. In the unauthenticated path: assert `find.text('Sign In')` exists (no change from current)

## Key Widgets to Assert

| Screen | Widget / Text to find |
|--------|----------------------|
| Tickets list | `find.text('Tickets')` (tab), ListView with items OR empty-state widget |
| Ticket detail | Tab labels: `'Notes'`, `'Charges'`, `'Fields'`, or `'Attachments'` |
| Back on list | `find.text('Tickets')` still visible in nav bar |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/tickets_test.dart \
  --flavor qa -d {device}
```

Expected: test passes on authenticated simulator with at least one assertion beyond tab tap.

---

## Notes

- Use `$.pump(const Duration(seconds: 3))` after tapping tab (list makes API call)
- The ticket list is paginated — don't assume items exist; always guard with `if ($(finder).exists)`
- `ListTile` may not be the exact widget type — inspect with Flutter inspector if needed; fallback to `find.descendant` patterns
