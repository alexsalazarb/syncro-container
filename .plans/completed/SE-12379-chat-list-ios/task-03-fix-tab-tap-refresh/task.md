# Task: Fix Tab-Tap Refresh Guard After Chat Detail Return

**Plan**: SE-12379-chat-list-ios
**Task ID**: task-03
**Task Path**: task-03-fix-tab-tap-refresh
**Depends On**: None
**Ticket**: SE-12379

## Objective

Fix the Chats tab-tap handler in `HomePage` so it triggers `ChatCubit.refresh()` even when the user is returning from a chat detail page (where the current route is `/chat-detail`, not `/chat`).

## Context

See [investigation.md](../investigation.md) Root Cause 3.

`lib/features/home/presentation/home_page.dart:133`:

```dart
if (currentRoute == AppRoute.chats.path) {
  context.read<ChatCubit>().refresh();
}
```

The chat detail is pushed on `_rootNavigatorKey`, so when the user is on chat detail the shell's current route is still `/chat-detail` (not `/chat`). When the user taps the Chats tab, the equality check fails and no refresh is triggered.

The fix is to broaden the condition to cover the chats branch — check that the current route starts with `/chat` OR that the user is re-selecting the chats branch regardless of sub-route.

## Before You Start

- [ ] Switch to base branch and pull latest: `git switch main && git pull --rebase origin main`
- [ ] Read the ticket: https://syncrotech.atlassian.net/browse/SE-12379
- [ ] Read [investigation.md](../investigation.md) Root Cause 3
- [ ] Check `status.md` — if already `in-progress` or `complete`, stop and investigate
- [ ] Read the full tab-tap handling section in `lib/features/home/presentation/home_page.dart` (around line 90–175) before writing
- [ ] Mark this task `in-progress` in `status.md` before proceeding

## File Ownership

| File | Action | Notes |
|------|--------|-------|
| `lib/features/home/presentation/home_page.dart` | modify | Fix chats tab-tap guard condition only |

### Do NOT Modify

- `lib/core/services/chat_websocket_service.dart` — owned by task-01
- `lib/features/chat/application/new_chat_cubit.dart` — owned by task-02
- `lib/features/chat/application/chat_cubit.dart` — owned by task-04

## Implementation Steps

### Step 1: Read the full tab-tap section

Open `lib/features/home/presentation/home_page.dart` and read the entire block that handles tab taps (roughly lines 90–175). Understand the pattern used for ALL tabs — home, appointments, chats, assets, tickets. Make sure your fix follows the same style as the others.

### Step 2: Fix the chats condition

Locate the chats block (around line 133):

```dart
final currentRoute = context
    .read<RouterStateNotifier>()
    .currentRoute;  // (verify exact accessor)

if (currentRoute == AppRoute.chats.path) {
  context.read<ChatCubit>().refresh();
}
```

Change the equality check to a `startsWith` check:

```dart
if (currentRoute.startsWith(AppRoute.chats.path)) {
  context.read<ChatCubit>().refresh();
}
```

`AppRoute.chats.path` is `/chat`. `startsWith('/chat')` covers `/chat` itself AND `/chat-detail` (and any future sub-routes). This is consistent with how the chats branch is structured.

**Important**: verify that this doesn't accidentally match unrelated routes. `/chat` is a unique enough prefix in the route list — check `routes_names.dart` to confirm no other routes start with `/chat` that shouldn't trigger a refresh.

### Step 3: Format and analyze

```bash
cd syncro-flutter
fvm dart format lib/features/home/presentation/home_page.dart
fvm flutter analyze
```

## Testing

- [ ] `fvm flutter test` passes (all existing tests green)
- [ ] `pre-commit-check` passes
- [ ] No analyzer warnings

## Documentation / KB Updates

- [ ] No KB/doc updates required.

## Completion Criteria

- [ ] Chats tab-tap guard uses `startsWith(AppRoute.chats.path)` instead of `==`
- [ ] No other tabs are affected by the change
- [ ] `fvm flutter analyze` clean
- [ ] All existing tests pass
- [ ] Changes committed to `plan/SE-12379-chat-list-ios/task-03-fix-tab-tap-refresh` branch
- [ ] Status updated in `status.md`
