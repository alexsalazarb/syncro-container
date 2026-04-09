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

<!-- CUSTOMIZE: Fill in your project details -->
**[Project Name]**: Brief description of what this project does.

| Aspect | Value |
|--------|-------|
| Platform | Flutter (iOS + Android <!-- + Web, Desktop if applicable -->) |
| Language | Dart 3.x |
| Flutter SDK | 3.x <!-- adjust as needed --> |
| State Management | <!-- Riverpod, BLoC, Provider, GetX --> |
| Navigation | <!-- GoRouter, Navigator 2.0, AutoRoute --> |
| DI / Service Locator | <!-- get_it, Riverpod, Injectable --> |
| Main branch | `main` |
<!-- Optional: | Work Plans Path | /path/to/work-plans | Hub for master plans (also configurable via WORK_PLANS_PATH in .ai-framework.config) | -->

**Key commands**:
```bash
# Run app
flutter run
flutter run --flavor production  # if flavors configured

# Build
flutter build apk                 # Android
flutter build ipa                 # iOS
flutter build web                 # Web

# Tests
flutter test                      # unit + widget tests
flutter test integration_test/    # integration tests

# Lint & Format
flutter analyze
dart format .

# Dependencies
flutter pub get
flutter pub upgrade
```

---

## Knowledge Base

### 3-Layer Loading (follow this order)

1. **Layer 1 — Framework KB**: Read `.ai-framework/knowledge-base/flutter/_index.md`. Match task keywords against Triggers. Load `.compact.md` first; escalate to full doc only if compact is insufficient.
2. **Layer 2 — Container KB**: Read `docs/kb-container/README.md` at repo root for cross-project domain docs (in single layout, same as Layer 3).
3. **Layer 3 — Project KB**: Read `docs/kb-container/README.md` for project-specific docs.

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

<!-- CUSTOMIZE: Adjust based on your project -->
1. **No `BuildContext` across async gaps** - Never use `context` after `await` without checking `mounted`
2. **Widget immutability** - Keep widgets immutable; move mutable state to `State` or state management layer
3. **`const` constructors** - Use `const` wherever possible to avoid unnecessary rebuilds
4. **Dispose resources** - Always dispose `AnimationController`, `TextEditingController`, `StreamSubscription`, etc. in `dispose()`
5. **Error handling** - Handle `Future` errors; don't let unhandled exceptions silently fail
6. **No hardcoded strings** - Use localization (`AppLocalizations` / `intl`) and constants
7. **Tests required** - Include unit tests, widget tests, and integration tests for all changes
8. **Platform-specific code** - Isolate in platform channels or conditional imports; don't scatter `Platform.isIOS` checks

---

## Flutter-Specific Patterns

<!-- CUSTOMIZE: Add your project's patterns -->

### Project Structure
```
lib/
  main.dart
  app/                  # App entry, router, theme
  features/             # Feature-first organization
    [feature]/
      data/             # Repositories, data sources, DTOs
      domain/           # Entities, use cases, interfaces
      presentation/     # Widgets, pages, state (BLoC/Riverpod)
  core/                 # Shared utilities, extensions, constants
    widgets/            # Reusable UI components
    theme/              # Colors, typography, spacing
    network/            # HTTP client setup
    storage/            # Local storage abstractions
  l10n/                 # Localization ARB files
test/
  unit/
  widget/
integration_test/
```

### Naming Conventions
- Classes/widgets: PascalCase (`UserProfilePage`, `AvatarWidget`)
- Functions/variables: camelCase (`fetchUser()`, `isLoggedIn`)
- Constants: lowerCamelCase or SCREAMING_SNAKE_CASE (`kPrimaryColor`, `MAX_RETRIES`)
- Files: snake_case (`user_profile_page.dart`)
- Test files: mirror source path with `_test.dart` suffix

### Widget Structure
```dart
class FeaturePage extends StatelessWidget {
  const FeaturePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Feature')),
      body: const _FeatureBody(),
    );
  }
}

class _FeatureBody extends StatelessWidget {
  const _FeatureBody();

  @override
  Widget build(BuildContext context) {
    // stateless inner widget
  }
}
```

### Async Context Safety
```dart
Future<void> _doSomething(BuildContext context) async {
  final result = await someAsyncCall();
  if (!context.mounted) return;  // Always check after await
  Navigator.of(context).pop(result);
}
```

---

## Things to Avoid

1. **Over-engineering** - Only make directly requested changes
2. **`BuildContext` after `await`** - Always check `mounted` first
3. **Stateful widgets for everything** - Prefer stateless + state management
4. **`setState` in deep widget trees** - Lift state up or use state management
5. **Missing `const`** - Add `const` to constructor calls where possible
6. **Forgetting `dispose()`** - Always clean up controllers and subscriptions
7. **Fat widgets** - Extract reusable components; keep `build()` readable
8. **Ignoring existing patterns** - Match the codebase's conventions

<!-- CUSTOMIZE: Add project-specific anti-patterns -->

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

<!-- CUSTOMIZE: Update with your project commands -->
| Task | Command |
|------|---------|
| Run app | `flutter run` |
| Unit/widget tests | `flutter test` |
| Integration tests | `flutter test integration_test/` |
| Analyze | `flutter analyze` |
| Format | `dart format .` |
| Build Android | `flutter build apk` |
| Build iOS | `flutter build ipa` |

---

## REMEMBER (End of File)

Before responding, check:
1. **Am I being corrected?** → Run `log-mistake`
2. **Is user committing/staging?** → Run `pre-commit-check`
3. **Is user ending session?** → Run `session-end-checklist`
4. **Did I solve something complex without KB?** → Run `document-solution`

Detect and execute automatically.
