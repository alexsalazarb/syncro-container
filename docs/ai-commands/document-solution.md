
# Document Solution

## Context Required
FULL-CONTEXT: AGENTS.md + relevant KB

## Triggers

### Automatic (CRITICAL - KB Fallback-Update Rule)

**KB Miss → MUST Document**: When this sequence occurs:
1. You checked KB for relevant docs
2. No relevant doc existed
3. You searched code/investigated to solve
4. You successfully solved the problem

→ **MUST run `document-solution`** to capture the pattern.

**Other automatic triggers:**
- Bug fix required understanding 3+ files to solve
- Solution used a non-obvious framework, library, or architecture pattern not in KB
- Conversation had 5+ back-and-forth exchanges to resolve
- Integration with an external service or third-party SDK
- Data migration or schema change was required

### Manual
- `/document-solution`
- "run document-solution"
- "document this"
- "add this to the knowledge base"

## Instructions

### Step 0: Resolve project context

Follow the algorithm in [`common/resolve-project-context.md`](../common/resolve-project-context.md):

1. Read `.ai-framework.config` → build `folder:stack` map from `PROJECTS` (or `PROJECT_DIR`/`STACK` for single layout).
2. Infer the active project from the files being discussed or edited (check path prefixes against the map). If ambiguous, ask.
3. Resolve **`CONTAINER_KB_ROOT`** from config (default `docs/kb-container`), then **`KB_DIR`** and **`CONTAINER_KB_DIR`** per §4 of [`resolve-project-context.md`](../common/resolve-project-context.md) (embedded vs isolated vs single).

> Project folder keys are arbitrary — do not assume they match the stack name.

### Step 1: Analyze the solution
- What was the root problem?
- What files/models were involved?
- What was the non-obvious part?
- What gotchas did we encounter?

### Step 2: Determine category and destination

**Per-project (Layer 3) — `KB_DIR`** (same subpaths as Layer 2; see `resolve-project-context` §5):

| Category | Path | Use when |
|----------|------|----------|
| Architecture / structure | `KB_DIR/technical/architecture/` | MVVM, navigation, module layout for this codebase |
| Stack API consumption | `KB_DIR/technical/api/` | How this stack calls the backend |
| BLE (platform notes) | `KB_DIR/technical/ble/` | Implementation notes not shared across platforms |
| Data shapes (local) | `KB_DIR/technical/data-models/` | DTOs / view models specific to this stack |
| Third-party SDKs | `KB_DIR/technical/integrations/` | StoreKit, Play Billing, etc. |
| Stack standards | `KB_DIR/engineering/best-practices/` | Swift/Kotlin/Python conventions for this app |
| Security (local) | `KB_DIR/engineering/security/` | Platform-specific security notes |
| Testing (local) | `KB_DIR/engineering/testing/` | Stack test patterns |
| Agent meta-pattern | `KB_DIR/ai-patterns/` | Mistake log, known issues |
| Platform-only feature | `KB_DIR/product/features/` | Screens/capabilities only on this stack |

**Shared container (Layer 2) — `CONTAINER_KB_DIR`:**

| Category | Path | Use when |
|----------|------|----------|
| Domain concept / business rule | `CONTAINER_KB_DIR/product/domain/` | Concept exists across all platforms |
| Shared REST API contract | `CONTAINER_KB_DIR/technical/api/` | API consumed by multiple stacks |
| BLE protocol / packet spec | `CONTAINER_KB_DIR/technical/ble/` | BLE lifecycle, packet types |
| Canonical data model | `CONTAINER_KB_DIR/technical/data-models/` | Request/response shape across platforms |
| Cross-stack engineering standard | `CONTAINER_KB_DIR/engineering/best-practices/` | Convention applying regardless of stack |
| Security / compliance rule | `CONTAINER_KB_DIR/engineering/security/` | Auth policy, data handling |
| Cross-stack testing strategy | `CONTAINER_KB_DIR/engineering/testing/` | Test approach shared across stacks |
| Agent mistake or correction | `CONTAINER_KB_DIR/ai-patterns/mistake-log.md` | Pattern to avoid |

> **Decision rules:**
> - "Would an Android dev care?" → yes → Layer 2 (`CONTAINER_KB_DIR`)
> - "Is this about *what* the product does?" → `product/domain/`
> - "Is this about *how* platforms talk to each other?" → `technical/`
> - "Is this about *how we build* things?" → `engineering/`
> - "How does *this platform* present or implement this?" → `KB_DIR/product/features/`
> - "This feature doesn't exist on any other platform" → `KB_DIR/product/features/`

### Step 3: Generate filename
- Format: `kebab-case-topic.md`
- Example: `realm-migration-strategy.md`, `auth-token-refresh.md`, `retry-policy-pattern.md`

### Step 4: Create document

Follow this structure:

```markdown
# [Topic Title]

**Last Updated**: [Month Year]
**Context**: Read when [trigger conditions — specific keywords].

---

## Overview

[1-2 paragraphs explaining the problem space]

---

## The Problem

### Symptoms
- [How this manifests]

### Root Cause
[Why it happens]

---

## The Solution

### Key Files

| File | Role |
|------|------|
| `path/to/file` | [What it does] |

### Code Pattern

[The correct approach with a code example in the project's primary language]

---

## Gotchas

### [Gotcha 1]
[What to watch out for]

---

## Related

- [Link to related KB doc]
```

### Step 5: Save document
Save to `{destination}/{category}/{filename}.md` where `{destination}` is `KB_DIR` or `CONTAINER_KB_DIR` per the routing table in Step 2.

### Step 6: Run check-kb-index
Update the knowledge base index for the destination directory used in Step 5.

### Step 7: Hand off for review — separate KB PR

Do NOT commit. Show the written content, then tell the user to open a dedicated PR for the KB change. Follow §6c of [`common/resolve-project-context.md`](../common/resolve-project-context.md).

```
KB addition ready for review.

  File: {destination}/{category}/{filename}.md
  Layer: {2 — shared across all projects | 3 — project-specific: {folder}}

When you're satisfied with the content, open a dedicated PR for this KB change:

  git checkout -b kb/{topic-slug}
  git add {file} {index}
  git commit -m "docs(kb): add {filename}"
  git push -u origin kb/{topic-slug}
  # then open a PR targeting main

Keep KB PRs separate from plan files and application code.
Plans push directly to main. KB changes go through a PR so the team can review.
```

### Step 8: Framework contribution check (automatic)

After saving, evaluate whether the pattern is **generic enough to benefit all projects** of this stack — not just this project.

Ask yourself:
- Could a developer on a different project using the same stack hit this exact problem?
- Does the solution rely on project-specific context (business logic, custom classes), or is it pure stack knowledge?
- Is this pattern already in `.ai-framework/knowledge-base/{STACK}/`?

**If the pattern is generic**, proactively tell the user:

```text
This pattern looks reusable across {STACK} projects.
Consider contributing it to the framework: /contribute-pattern

This will also generate a compact variant and update _index.md
so future agents can find it via tiered loading.
```

**If the pattern is project-specific** (uses project classes, business logic, custom infra), skip this step silently.

> **Note**: When the pattern is contributed to the framework via `/contribute-pattern`,
> that skill will call `/add-kb-doc` to generate the compact variant and update the
> stack's `_index.md` routing table.

## Output

- New file: `{destination}/{category}/{filename}.md`
- Updated: `{destination}/README.md` (via check-kb-index)
- Confirmation with document location
- Framework contribution suggestion if pattern is generic
