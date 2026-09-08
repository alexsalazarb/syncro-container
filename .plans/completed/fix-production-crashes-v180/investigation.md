# Investigation: fix-production-crashes-v180

**Date**: 2026-09-07
**Trigger**: Full Crashlytics audit (iOS + Android, 90-day window) requested before planning any fix.

**Correction (2026-09-07, same day)**: This investigation originally covered 3 issues including a WebView auth-challenge crash (see retracted section below). While preparing to push, `git fetch` revealed `origin/main` has a commit (`3f1afaf`, 2026-08-28) not present on this branch (`POCAuth`): `.plans/1.7.1-crashlytics/`, a 6-task plan covering 8 Crashlytics issues — including the exact same issue ID this investigation targeted, with a different and better-evidenced root cause. See the retracted section for full detail on what was wrong and why.

**Merge (2026-09-07, later same day)**: The user asked whether the two plans could be combined. They were merged into this one (`fix-production-crashes-v180`) — see the Merge Log in [overview.md](overview.md). This investigation.md only covers what was originally planned here: `Ticket.fromJson` (now `task-07`) and Crashlytics housekeeping (now `task-08`). The 6 absorbed tasks (`task-01` through `task-06`) have their own root-cause detail written directly into each `task.md`'s Context section from the original `1.7.1-crashlytics` planning session — see overview.md's Root Cause Summary table for a one-line-per-task index into those.

## Method

1. Read `.plans/fix-production-crashes-v140/` (root, not archived) and `.plans/completed/fix-production-crashes-v152/` to check for existing coverage — see note below on a documentation gap found during this step.
2. Pulled `crashlytics_get_report` (topIssues, 90 days) for both production apps (`com.servably.syncro.mobile`, Android appId `...3352304fd0baa59e5b5c5b`, iOS appId `...a0d84c75923c83b35b5c5b`).
3. For each candidate issue, pulled `crashlytics_get_issue` + `crashlytics_batch_get_events` for a real stack trace, then cross-referenced against the current source with `rg`/`git blame`.

## Documentation gap found (not part of this plan's fix, noted for awareness)

`.plans/fix-production-crashes-v140/overview.md` says `Status: complete`, but all 4 of its `task-*/status.md` files say `not-started`. Verified via `git log` on the plan's Key Files table that all 4 fixes actually shipped in commit `1da741e2` ("SE-11997: Firebase Stability Improvements v1.4.0", 2026-04-20) — the overview was right, the task status files were simply never updated. Consequence: two of those same issues (asset-filter deserializer, chat-websocket connect null-check) were fixed a **second time** a month later under `fix-production-crashes-v152` (`SE-12498`/`SE-12499`) — duplicated work caused entirely by the stale task status. Also noted: `fix-production-crashes-v140` was never moved into `.plans/completed/` despite being done, even though the root `.plans/README.md` Completed Plans table already lists it as such.

**Takeaway applied to this investigation**: every root-cause claim below was verified against a real Crashlytics stack trace and the current state of the file, not just against a plan's stated status.

## Task 07 — `Ticket.fromJson` null cast

### Origin Analysis
`git blame` on `lib/features/ticket/ticket_home/domain/ticket.dart:52-59` shows the current unguarded casts were introduced by Alex Salazar on 2025-11-12 (commit `4f8ba837a`) — a refactor that added several other fields with proper `as Type?` null-safety (`status`, `priority`, `customer_id`, `customer_business_then_name`) but left `id` (line 53) and `number` (line 54) as hard `as int` casts.

### Real stack trace (Android, issue `f91fb1b40d29c211bdf65655b75937b8`)
```
io.flutter.plugins.firebase.crashlytics.FlutterError: type 'Null' is not a subtype of type 'int' in type cast.
at new Ticket.fromJson (ticket.dart:54)
at _parseTicketsResponse.<anonymous closure> (get_tickets_deserializer.dart:10)
at MappedIterable.elementAt (iterable.dart:402)
...
at compute.<anonymous closure> (_isolates_io.dart:23)
```
Confirms the crash is specifically the `number` cast on line 54, happening inside the `compute()` isolate used to parse the tickets list off the main thread — a failure here throws for the *entire page* of tickets, not just the offending row.

### Confirmed on both platforms
- Android: `f91fb1b40d29c211bdf65655b75937b8` — 2 events / 1 user, v1.7.0
- iOS: `50306dfd3ee04e4d8c71ae4d212954dc` — 6 events / 2 users, v1.6.1 (event sample confirms identical `ticket.dart:54` frame)

### Test Coverage Analysis
`test/features/ticket/ticket_home/domain/ticket_test.dart` has a `fromJson handles null user` test but never feeds a JSON map with `id: null` or `number: null` — the exact gap that let this ship. The existing "happy path" and "null user" tests both always supply valid `id`/`number` values.

### Precedent
Same shape as the 4 bugs already fixed under `SE-11997` (`fix-production-crashes-v140`): `CustomerInformation.fromJson` (`lib/features/assets/domain/get_chat_information_by_ids_response.dart`) already uses `(json['fullname'] as String?) ?? ''` for exactly this reason. `GetTicketsSettingsDeserializer.fromJson` uses a `data is! Map` guard + `FirebaseCrashlytics.instance.recordError(..., fatal: false)` pattern worth reusing here if a non-fatal report is desired instead of silent defaulting.

### Confidence: **HIGH**
Real stack trace pinpoints the exact line and field; the fix (null-safe cast + regression test) is mechanical and matches an already-established repo pattern.

---

## WebView auth-challenge dispose race (RETRACTED hypothesis — see below)

> **This task was removed from the plan.** Kept here, struck through in spirit, for the record — it's a real example of an investigation that reached HIGH confidence on an incorrect root cause, and *why* it was wrong is worth remembering.

### What I originally concluded
- `_clearWebViewCache()` call in `dispose()` (`login_web_view.dart:154-159`, introduced 2025-07-10, commit `ac4b4dde5`) races the native `WKWebView` teardown against a pending HTTP auth challenge, causing `NSInternalInconsistencyException` during Swift ARC dealloc for Crashlytics issue `b0914a639eb5e4b996dec3e8f1147a4b` (iOS, FATAL, firstSeen v1.7.0).
- I verified "no missing handler elsewhere" via `rg -l "NavigationDelegate\(" lib/` and `rg -l "onHttpAuthRequest" lib/` — both returned only `login_web_view.dart`, which I read as "there's exactly one WebView in the app and it has the handler."

### Why that was wrong
That grep only finds files that **construct** a `NavigationDelegate` — it cannot find a WebView that has **no delegate at all**, which is exactly the actual bug. `origin/main` commit `3f1afaf` (a plan called `.plans/1.7.1-crashlytics/`, not present on this branch at investigation time because `POCAuth` diverged from `main` before it landed) already root-caused the *same issue ID* correctly: `lib/features/ticket/ticket_attachment/presentation/attachment_preview_view.dart:44` constructs `controller: WebViewController()` — the bare constructor, not `.fromPlatformCreationParams(...)` like `login_web_view.dart` — and **never calls `.setNavigationDelegate(...)` at all**. Re-running `rg -n "WebViewController\(\)" lib/` (searching for the constructor call itself, not for `NavigationDelegate`) immediately surfaces this file and only this file. The actual bug is "second WebView, zero auth-challenge handling," not "first WebView, handled but racing disposal." My hypothesis about `dispose()`/`_clearWebViewCache()` may still describe a real code smell, but it isn't demonstrated to be *this* crash's cause, and shouldn't be treated as one without separate evidence.

### Lesson
When a `NavigationDelegate` (or any config object) is the suspected missing piece, grep for the *thing that should have the config* (every `WebViewController(...)` call site, every constructor pattern) — not just for the config keyword itself. A search for "X" can never find "the place X is absent."

### Disposition
Issue `b0914a639eb5e4b996dec3e8f1147a4b` is owned by `phase-2/task-05-webview-attachment-auth-challenge/` in this same plan (merged in — see overview.md Merge Log). Its `task.md` already has the correct root cause; nothing more to add here.

---

## Task 08 — Crashlytics housekeeping

Straightforward: `crashlytics_get_report` shows `lastSeenVersion` for these 3 issues predates the current release (1.8.0) by 2-4 versions, with the underlying code fix independently verified in git history (`SE-12498`, `SE-12500`) or explicitly out of scope for the app (backend 504s). No code investigation needed — just close them via `crashlytics_update_issue` so future Crashlytics triage isn't misled by stale OPEN issues.

## Cross-Project Assessment

Single-project bug — both fixes are entirely within `syncro-flutter`. No backend or cross-repo dependency.
