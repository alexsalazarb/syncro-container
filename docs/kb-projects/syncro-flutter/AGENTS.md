# AGENTS.md - AI Agent Instructions

Cursor, Codex, Windsurf, and Copilot read this file directly. Claude Code uses `CLAUDE.md`, Antigravity uses `.agent/rules/agent.md` as shims.

---

## QUICK TRIGGERS (Memorize These)

**Session Start**: Check `docs/kb-container/ai-patterns/mistake-log.md` for patterns to avoid.

**During Session**:
- User says "commit/stage" → run `pre-commit-check`
- You get corrected → run `log-mistake`
- KB lookup fails → after solving, run `document-solution`
- You modify KB files → run `check-kb-index`

**Session End** (user says "done/thanks/bye"): Run `session-end-checklist`

---

## Project Context

**Syncro Flutter**: MSP field-service mobile app for technicians — manage tickets, assets, real-time chat, appointments and time tracking on iOS and Android.

| Aspect | Value |
|--------|-------|
| Platform | Flutter (iOS + Android) |
| Language | Dart 3.x (`sdk: ">=3.8.0 <4.0.0"`) |
| Flutter SDK | 3.x |
| App Version | 1.4.0+412 |
| State Management | `flutter_bloc` 9.x — Cubit-first |
| Navigation | `go_router` 14.x — `StatefulShellRoute.indexedStack` (5 bottom tabs) |
| DI / Service Locator | `get_it` 8.x (singletons) + `BlocProvider`/`RepositoryProvider` (widget-tree scoped) |
| HTTP | `dio` 5.x via `NetworkService` abstraction |
| Local DB | `hive` 2.x (via `LocalDBService`) + `shared_preferences` |
| Real-time | `phoenix_socket` — `ChatWebSocketService` singleton |
| Auth | OAuth2 via `oauth_webauth` + `flutter_secure_storage` |
| Error Handling | `dartz` `Either<Failure, T>` throughout all layers |
| Main branch | `main` |
| Testing | `flutter_test` + `mockito` + `bloc_test` |

**Key commands** (project uses fvm — always prefix with `fvm`):
```bash
# Run app
fvm flutter run
fvm flutter run --dart-define=FLAVOR=qa  # QA environment

# Build
fvm flutter build apk                     # Android
fvm flutter build ipa                     # iOS

# Tests
fvm flutter test                          # unit + widget tests
fvm flutter test --coverage               # with coverage (see generate_coverage.sh)

# Code generation (mocks)
fvm flutter pub run build_runner build --delete-conflicting-outputs

# Lint & Format
fvm flutter analyze
fvm dart format .

# Dependencies
fvm flutter pub get
```

---

## Knowledge Base

### 3-Layer Loading (follow this order)

1. **Layer 1 — Framework KB**: Read `.ai-framework/knowledge-base/flutter/_index.md`. Match task keywords against Triggers. Load `.compact.md` first; escalate to full doc only if compact is insufficient.
2. **Layer 2 — Container KB**: Read `docs/kb-container/README.md` at repo root — product domain concepts (ticket, asset, customer, interaction).
3. **Layer 3 — Project KB**: Read `docs/kb-projects/syncro-flutter/README.md` — architecture patterns, integrations, features, known issues.

Load ONLY relevant docs. Do not load entire KB.

### KB-First Rule

Before exploring code in any area, check KB across all three layers:

1. Check Layer 1 `_index.md` triggers for the topic
2. Check Layer 2/3 `README.md` indexes
3. If a matching KB doc exists → read it before any grep/read operations on that code
4. Only explore code directly if no KB doc covers the area
5. If you explore code directly and discover important patterns → create KB documentation using `document-solution`
6. If you modify functionality that has existing KB docs → update those docs to reflect the changes

This applies mid-task too. If you start investigating a different subsystem, re-check the KB indexes before exploring that code.

**Why**: KB docs contain curated knowledge that prevents unnecessary code exploration.

### Optional: technology choices per topic

Projects do **not** need this on day one. When you standardize (e.g. Riverpod vs Provider, isolates vs compute), add Layer 3 docs and list them here so agents load the right framework guides and skip conflicting ones.

See `.ai-framework/docs/KB_TOPIC_STANDARDS.md` for the supported shapes (per-topic files, one routing table, or AGENTS-only).

<!-- CUSTOMIZE (optional): When ready, uncomment and link your Layer 3 docs -->
<!-- - State / DI: {KB_DIR}/technical/architecture/state-and-di-standard.md -->
<!-- - Persistence: {KB_DIR}/technical/architecture/persistence-standard.md -->
<!-- - Or single table: {KB_DIR}/technical/architecture/technology-choices.md -->

---

## Critical Constraints

1. **No `BuildContext` across async gaps** - Never use `context` after `await` without checking `mounted`
2. **Use `safeEmit()` not `emit()`** - Always use `safeEmit()` from `core/utils/cubit_extension.dart` in cubits
3. **Use `logger()` not `print`/`debugPrint`** - Always import from `core/utils/logger.dart`
4. **`Either<Failure, T>` for all repo calls** - Never throw from repositories; always return `Either`
5. **Never use `NetworkService` from feature widgets** - Always go through repository → use case → cubit
6. **Don't create new DI registrations in GetIt for feature repos** - Use widget-tree `RepositoryProvider` instead
7. **Const constructors** - Use `const` wherever possible to avoid unnecessary rebuilds
8. **Dispose resources** - Always dispose `AnimationController`, `StreamSubscription`, etc. in `dispose()`
9. **Token refresh is not implemented** - Do not rely on automatic token refresh (known critical issue — see `ai-patterns/known-issues.md`)

---

## Flutter-Specific Patterns

### Actual Project Structure
```
lib/
  main.dart                 # App init pipeline (Firebase → Hive → GetIt → Notifications)
  app/
    dependency/
      service_locator.dart  # GetIt singleton registrations
      app_repositories.dart # RepositoryProvider registrations
      app_providers.dart    # BlocProvider registrations
    view/
      app.dart              # MaterialApp.router
  core/
    configs/                # Environment (multi-env via .env files)
    design_system/themes/   # ThemeCubit
    global_widgets/         # atoms/, molecules/, custom fields
    networking/             # NetworkService → Dio (rest_network_service.dart)
    routing/                # RouteCubit + GoRouter (app_router.dart)
    services/               # Analytics, Notifications, RemoteConfig, Hive, WebSocket, Storage
    usecases/               # Abstract UseCase base classes
    utils/                  # logger, SharedPreferences, extensions
  features/                 # 17 features (alerts, appointments, asset_detail, assets,
                            # authentication, chat, chat_detail, customers, dashboard,
                            # end_user, home, settings, splash, technicians, ticket,
                            # time_clock, unlock_app)
test/
  core/
  features/
```

### Feature Layer Convention
```
features/[feature]/
  domain/         # Entities, params, repository interfaces, response types
  infrastructure/ # RepositoryImpl, UseCases (⚠️ use cases here, not domain/)
  application/    # Cubits (state management)
  presentation/   # Pages, widgets
```

### Naming Conventions
- Classes/widgets: PascalCase (`TicketDetailsPage`, `TicketCardWidget`)
- Cubits: `PascalCase` + `Cubit` (`TicketCubit`)
- Repository interfaces: `PascalCase` + `Repository` (`TicketsRepository`)
- Repository impls: `PascalCase` + `RepositoryImpl` (`TicketsRepositoryImpl`)
- Use cases: `PascalCase` + `UseCase` (`GetTicketsUseCase`)
- Params: `PascalCase` + `Params` (`GetTicketsParams`)
- Files: snake_case (`ticket_details_page.dart`)

### Key Patterns to Follow
- `safeEmit()` instead of `emit()` in all cubits
- `Either<Failure, T>` for all async results from repositories
- `logger()` from `core/utils/logger.dart` — never `print()`/`debugPrint()`
- `_createTypedRoute<T>()` for type-safe GoRouter parameters
- Feature flags via `FeatureFlagManagerImpl()` — never access RemoteConfig directly
- `unawaited()` for fire-and-forget service calls (Pendo, chat socket init)

### Patterns to Avoid
- Double DI registration (GetIt AND RepositoryProvider for same repo)
- Creating new GetIt registrations for feature-level repositories
- Passing `BuildContext` across `await` without checking `mounted`
- Using `emit()` directly in cubits (use `safeEmit()`)
- Directory names with spaces (e.g., `edit custom fields/` — use `edit_custom_fields/`)

---

## Things to Avoid

1. **Over-engineering** - Only make directly requested changes
2. **`BuildContext` after `await`** - Always check `mounted` first
3. **`emit()` directly** - Use `safeEmit()` instead
4. **`print()` / `debugPrint()`** - Use `logger()` from `core/utils/logger.dart`
5. **Stateful widgets for everything** - Prefer stateless + Cubit state management
6. **Missing `const`** - Add `const` to constructor calls where possible
7. **Forgetting `dispose()`** - Always clean up controllers and subscriptions
8. **Fat widgets** - Extract reusable components; keep `build()` readable
9. **Registering repos in GetIt** - Use widget-tree `RepositoryProvider` for feature repos
10. **Throwing from repositories** - Return `Left(Failure(...))` instead
11. **Spaces in directory names** - Always use underscores

---

## Mandatory Auto-Triggers

These MUST fire automatically. Do not wait for user to ask:

| Trigger | Action | Why |
|---------|--------|-----|
| User says "commit", "stage", "git add" | `pre-commit-check` | Catch violations |
| KB lookup fails → solved via code | `document-solution` | Update KB |
| User corrects you | `log-mistake` | Build patterns |
| Modified KB files | `check-kb-index` | Keep index current |
| User ending session | `session-end-checklist` | Catch missed automation |

### Trigger Detection Examples

```
USER: "let's commit these changes"
→ Detected "commit" → run pre-commit-check BEFORE git operations

USER: "no that's wrong, it should be..."
→ Detected correction phrase → run log-mistake

USER: "thanks, that's all for today"
→ Detected session end → run session-end-checklist
```

**Correction phrases** (run `log-mistake` immediately):
"that's wrong", "actually...", "no, it should be...", "you forgot to...", "you missed..."

Do NOT wait for explicit requests - detect and trigger proactively.

---

## Skills & Agents

Skills: `.agents/skills/` | `.claude/skills/`
Agents: `.claude/agents/` (orchestrator, planner, implementer, reviewer, researcher, documenter, tester, test-writer)

| Skill | When |
|-------|------|
| `pre-commit-check` | Before commit/staging |
| `session-end-checklist` | Session ending |
| `log-mistake` | User corrects you |
| `document-solution` | Complex problem solved or KB miss |
| `check-kb-index` | After KB file changes |
| `save-session` | Long session (20+ turns) |
| `check-test-coverage` | After implementing feature/fix |
| `check-agent-drift` | Periodic / on request |
| `cleanup-sessions` | Manual / maintenance |
| `list-skills` | Discover available automation |
| `create-plan` | Starting a new feature/project |
| `create-bug-plan` | Investigating + planning a production bug fix |
| `add-defect` | QA/reviewer adds a bug to an existing plan |
| `execute-task` | Working on a specific plan task |
| `execute-plan` | Running remaining plan tasks (parallel agents) |
| `manage-contracts` | Managing cross-repo interface contracts |
| `archive-plan` | Archiving completed plans and cleaning up linked repo plans |
| `transition-plan` | Moving plan between lifecycle states (promote, deprioritize, cancel) |
| `create-master-plan` | Creating cross-repo master plans in the work-plans repo |
| `sync-master-plan` | Syncing repo plan status to linked master plan |
| `plans-status` | Aggregated plan status dashboard |
| `pr-cleanup` | Automated CodeRabbit triage and CI polling for PRs |
| `upgrade-framework` | AI-assisted framework upgrade with intelligent skill review |
*Invoke skills with `/skill-name` or natural language. Legacy commands: `docs/ai-commands/`*

### Hub Commands (cross-repo master plans)
- `/create-master-plan` — Create a product-focused master plan in the work-plans hub
- `/transition-plan {slug} --to {state}` — Move master plan between lifecycle states
- `/sync-master-plan` — Sync local plan progress to hub
- `/plans-status` — Aggregated plan status dashboard

---

## Self-Documentation Rules

### Update KB when:
- KB lookup failed but you solved via code search → run `document-solution`
- Complex solution (3+ files, 5+ exchanges) → run `document-solution`
- After any KB change → run `check-kb-index`

### Format:
- Kebab-case filenames
- Include "Last Updated" date
- Include "Context" section

---

## Quick Reference

| Task | Command |
|------|---------|
| Run app | `fvm flutter run` |
| Unit/widget tests | `fvm flutter test` |
| Generate mocks | `fvm flutter pub run build_runner build --delete-conflicting-outputs` |
| Analyze | `fvm flutter analyze` |
| Format | `fvm dart format .` |
| Build Android | `fvm flutter build apk` |
| Build iOS | `fvm flutter build ipa` |
| Coverage report | `bash generate_coverage.sh` |

---

## REMEMBER (End of File)

Before responding, check:
1. **Am I being corrected?** → Run `log-mistake`
2. **Is user committing/staging?** → Run `pre-commit-check`
3. **Is user ending session?** → Run `session-end-checklist`
4. **Did I solve something complex without KB?** → Run `document-solution`

Detect and execute automatically.
