# KB Generation Rules

Rules and templates for Phase 6 of the `audit-and-plan` skill.
Governs where KB docs are written and what they contain.

---

## Three-Tier KB Model

The framework's `knowledge-base/` has three tiers:

| Tier | Location | Seeded on init? | When to copy to project KB |
|------|----------|----------------|---------------------------|
| **Universal standards** | `knowledge-base/{stack}/best-practices/` | ✅ Always | Already there — review and keep |
| **Architecture/testing patterns** | `knowledge-base/{stack}/architecture/` | ❌ No | Copy if the project follows this pattern |
| **Library/tool patterns** | `knowledge-base/{stack}/integrations/` | ❌ No | Copy only if the project uses that library/tool |

During this phase:
- **Best-practices docs** are already in `{KB_DIR}/engineering/best-practices/` — review them and note any project exceptions
- **Architecture/pattern docs**: if the project uses a pattern covered in the reference library (e.g. async/await, XCTest), copy the relevant doc from `.ai-framework/knowledge-base/{stack}/` into `{KB_DIR}/technical/architecture/`
- **Library docs**: if the project uses RxSwift → copy `rxswift-patterns.md`; if it uses Realm → copy `realm-patterns.md`; etc. Remove any stubs that don't apply.

---

## Per-Project Docs → `{KB_DIR}/`

Write once per project being audited. These describe *how* each stack works.

| Doc | Category | Content |
|-----|----------|---------|
| `project-overview.md` | `technical/architecture/` | Stack, entry points, top-level structure, module breakdown |
| `architecture-patterns.md` | `technical/architecture/` | Architecture style, base classes, data flow, DI, navigation/routing |
| `coding-conventions.md` | `technical/architecture/` | File org, naming, error handling, async pattern, "what a standard unit looks like" |
| One doc per platform-specific integration | `technical/integrations/` | e.g. StoreKit, Firebase on iOS; ActiveRecord patterns on Rails |
| `known-issues.md` | `ai-patterns/` | Problems found with severity (brief for Class A) |

---

## Shared Docs → `{CONTAINER_KB_DIR}/`

Write once for the container. These describe *what* the product does and what is shared.

| Doc | Category | Content |
|-----|----------|---------|
| `product-overview.md` | `product/domain/` | What the product does, who uses it, platform matrix, domain glossary |
| One doc per core domain concept | `product/domain/` | e.g. `user-model.md`, `subscription-flow.md`, `article-lifecycle.md` |
| One doc per shared REST API | `technical/api/` | e.g. the REST API consumed by iOS + Android, the auth provider |
| One doc per BLE protocol spec | `technical/ble/` | BLE lifecycle, packet types, connection flow |
| One doc per canonical data shape | `technical/data-models/` | Request/response shapes shared across platforms |
| Cross-stack engineering standard | `engineering/best-practices/` | Conventions applying regardless of stack |
| Security/compliance rule | `engineering/security/` | Auth policy, data handling requirements |

**Platform matrix** in `product-overview.md`:
```markdown
| Feature          | iOS | Android | Web (Admin) |
|------------------|-----|---------|-------------|
| Login            | ✓   | ✓       | ✓           |
| Content feed     | ✓   | ✓       | —           |
| Admin dashboard  | —   | —       | ✓           |
| Push notifs      | ✓   | ✓       | —           |
```

Each domain doc should note per-stack implementation differences inline:
```markdown
## iOS
Uses Combine to observe subscription state...
## Backend
Validates eligibility at /subscriptions endpoint...
```

---

## REQUIRED: Per-Project Platform Feature Docs → `{KB_DIR}/product/features/`

> **⚠ This section is frequently skipped by agents.** Every project MUST have at least
> one platform feature doc if the project has any UI, platform-specific behavior, or
> hardware interaction. Check the Phase 4b platform features list and create one doc
> per item.

For features or screens that are deeply platform-specific, or where each platform
implements a shared concept with meaningfully different libraries, permissions, or UI:

| Doc | Category | Content |
|-----|----------|---------|
| One doc per platform feature | `product/features/` | What this feature does on this platform, key files, business rules, UX notes |

**Example filenames** (isolated layout):
- `docs/kb-projects/ios/product/features/ble-pairing.md` — BLE device pairing flow on iOS
- `docs/kb-projects/ios/product/features/snore-recording.md` — audio recording pipeline
- `docs/kb-projects/ios/product/features/therapy-session.md` — therapy session UI + ViewModel
- `docs/kb-projects/android/product/features/treatment-service.md` — foreground service for therapy
- `docs/kb-projects/android/product/features/ble-pairing.md` — Nordic BLE scanner + Android permissions
- `docs/kb-projects/be/product/features/clinical-portal.md` — portal user roles, physician invitation
- `docs/kb-projects/be/product/features/background-jobs.md` — Dramatiq tasks, APScheduler

**Example filenames** (embedded layout):
- `ios/docs/kb-project/product/features/login-screen.md`
- `web/docs/kb-project/product/features/admin-dashboard.md`

---

## KB Doc Template

Use this template for each doc:

```markdown
# {Title}

**Last Updated**: {Month Year}
**Context**: Read when {specific trigger — keywords that should load this doc}.

---

## Overview

{1-2 paragraphs: what this is and why it matters}

---

## How It Works

{The actual logic / architecture / pattern. For feature docs: the user flow and business rules.}

---

## Key Files

| File | Role |
|------|------|
| `path/to/file` | What it does |

---

## Patterns & Conventions

{Recurring patterns an agent must follow when working in this area.
For architecture docs: the rules. For feature docs: business constraints.}

---

## Known Issues / Watch Out For

{Problems or gotchas. Brief for Class A; detailed for Class C.}
```
