# task-05 — Assets: list + detail + script output

| Field | Value |
|-------|-------|
| **Phase** | 2 |
| **Branch** | `plan/patrol-deep-tests/phase-2/task-05-assets-deep` |
| **Routes covered** | `assets`, `assetDetails`, `scriptOutput`, `addToQueue` |
| **Depends on** | None |

---

## Objective

Replace the shallow `assets_test.dart` stub. Navigate from Assets/Alerts tab into an asset detail screen, verify key widgets, and verify the script output screen opens from detail.

---

## File Ownership

**Modify:**
- `integration_test/patrol/features/assets_test.dart`

---

## Implementation Steps

1. In the authenticated path:
   a. Check if `'Assets'` tab exists (feature-flag-dependent); if not, check `'Alerts'` tab
   b. Tap the available tab, wait 3s
   c. Assert list renders with items or empty state
   d. If items exist: tap first item → verify `AssetDetailPage` (find asset name, type, or serial number field)
   e. From asset detail: look for a "Scripts" or "Run Script" option — if present, tap it and verify `scriptOutput` route opens (a text output widget or script list)
   f. Navigate back to list

## Key Widgets to Assert

| Screen | Find |
|--------|------|
| Assets list | `find.text('Assets')` or `find.text('Alerts')` + list |
| Asset detail | Asset name text, `find.text('Scripts')` or serial field |
| Script output | `find.text('Script Output')` or output text widget |

## Testing Verification

```bash
fvm flutter test integration_test/patrol/features/assets_test.dart \
  --flavor qa -d {device}
```

---

## Notes

- The Assets/Alerts tab swap is feature-flag controlled — guard with `if ($(find.text('Assets')).exists)` then try `find.text('Alerts')` as fallback
- `addToQueue` may only appear on asset detail if the account has RMM features enabled
