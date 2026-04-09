---
name: add-kb-doc
description: >-
  Adds a knowledge base document to the framework KB for any technology stack.
  Creates the full doc, auto-generates a compact variant if over 100 lines,
  and updates the stack's _index.md and README.md. Use when adding new
  technology patterns, when an agent escalates through all KB tiers and still
  has a gap, or after contribute-pattern promotes a project doc to the framework.
---

# Add KB Doc

## Context Required
LOW-CONTEXT: AGENTS.md, the knowledge to capture

## Triggers

### Automatic (KB Escalation Rule)

**Escalation gap**: When an agent follows the tiered loading protocol and:
1. Read `_index.md` — no matching doc for the topic
2. Searched code and/or web to find the answer
3. Successfully resolved the problem

→ **MUST run `add-kb-doc`** to fill the gap so future agents don't repeat the search.

**After `contribute-pattern`**: When a project KB doc is promoted to the framework,
run `add-kb-doc` to generate the compact variant and update `_index.md`.

### Manual
- `/add-kb-doc`
- "add this to the framework KB"
- "create a KB doc for [topic]"
- "add [technology] knowledge base"
- "document this pattern in the framework"

## Which Layer?

This skill writes **exclusively to Layer 1 (Framework KB)** — technology patterns that are
true for ALL projects of a given stack.

| Layer | Skill | Path |
|-------|-------|------|
| **Layer 1 — Framework** | **`add-kb-doc` (this skill)** | `.ai-framework/knowledge-base/{STACK}/` |
| Layer 2 — Container | `document-solution` | `docs/kb-container/` at container root |
| Layer 3 — Project | `document-solution` | `{PROJECT_CONTEXT_ROOT}` |

**Decision rule**: If the knowledge only applies to THIS project or THIS product, use
`document-solution` instead. `add-kb-doc` is for stack-universal patterns.

## Instructions

### Step 0: Detect operating mode and resolve context

**Detect which mode you are in:**

#### Framework-repo mode (no `.ai-framework.config`)

If `.ai-framework.config` does **not** exist at the repo root, check whether `knowledge-base/`
exists at the repo root. If it does, you are running **inside the framework repo itself**.

1. **Determine the stack** — either from a `--stack` argument or by asking which stack.
   Available stacks: list subdirectories of `knowledge-base/`.
2. **Validate** the chosen stack: confirm `knowledge-base/{stack}/` exists.
   If it doesn't, create it (see [new stack scaffold](references/new-stack-scaffold.md)) or ask.
3. Set `FRAMEWORK_KB_ROOT = knowledge-base/{stack}/`

#### Consumer-repo mode (`.ai-framework.config` present)

Read `.ai-framework.config` to get `STACK`.

Resolve `FRAMEWORK_KB_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md):
- **Consumer repo**: `.ai-framework/knowledge-base/{STACK}/`
- **Framework repo**: `knowledge-base/{STACK}/`

If the stack directory doesn't exist, create it per [new stack scaffold](references/new-stack-scaffold.md).

### Step 1: Gather the knowledge

**If triggered by escalation** (automatic): Use the context already available from the
agent's investigation — the problem, solution, files involved, and patterns discovered.

**If triggered manually**: Ask the user:
- "What technology or pattern does this cover?"
- "Which stack does it belong to?" (if not obvious from context)
- "What problem does it solve?"
- "When should a future agent load this doc?" (trigger keywords)

**If triggered after `contribute-pattern`**: Use the doc content already written.

### Step 2: Determine category and filename

Route using the framework KB structure:

| Category | When to use | Example filename |
|----------|------------|------------------|
| `best-practices/` | Universal rules — append to `{stack}-standards.md` | (append, don't create new file) |
| `architecture/` | Design patterns, data flows, concurrency, testing | `coordinator-pattern.md` |
| `ui-patterns/` | UI framework patterns, code style, view composition | `swiftui-navigation.md` |
| `integrations/` | Third-party library patterns (only if project uses it) | `combine-patterns.md` |
| `ai-patterns/` | Agent-specific rules, pre-commit checks | `pre-commit-react-rules.md` |

Filename format: `kebab-case-topic.md`

### Step 2b: Topic-standard awareness

After choosing the category, check whether this doc covers a **technology-choice topic**.
→ See [topic-standard awareness rules](references/kb-doc-template.md#topic-standard-awareness)

### Step 3: Check for duplicates

Before writing, scan the target directory for existing docs on the same topic.

- **Exact match exists** → update the existing doc instead. Go to Step 3U.
- **Related doc exists** → add a section to the existing doc. Go to Step 3U.
- **No match** → proceed to Step 4.

#### Step 3U: Update existing doc

1. Read the existing doc
2. Identify where the new content fits
3. Edit the doc
4. If the doc has a `.compact.md` variant, update that too
5. Skip to Step 6

### Step 4: Write the full document

→ See [KB document template](references/kb-doc-template.md#document-template)

Save to `knowledge-base/{STACK}/{category}/{filename}.md`.

### Step 5: Generate compact variant (if over 100 lines)

→ See [compact generation rules](references/compact-generation-rules.md#compact-variant-rules)

### Step 6: Update `_index.md`

→ See [index update format and tier rules](references/compact-generation-rules.md#index-update-rules)

### Step 7: Update `README.md`

Read `knowledge-base/{STACK}/README.md` and add the doc to the catalog table in the
appropriate category section. If the category section doesn't exist, create one.

### Step 8: Report

```text
KB Doc Added: knowledge-base/{STACK}/{category}/{filename}.md ({lines} lines)
Compact:     knowledge-base/{STACK}/{category}/{filename}.compact.md ({lines} lines)
Index:       knowledge-base/{STACK}/_index.md updated
README:      knowledge-base/{STACK}/README.md updated

Trigger keywords: {keywords from _index.md entry}
Future agents will auto-load this when working on: {brief description}
```

**Framework-repo mode**: commit with `git commit -m "feat({stack}): add {topic} to KB"`

**Consumer-repo mode**: commit inside `.ai-framework/` submodule with same message.

## Output

- New doc: `knowledge-base/{STACK}/{category}/{filename}.md`
- Compact variant (if >= 100 lines): `knowledge-base/{STACK}/{category}/{filename}.compact.md`
- Updated: `knowledge-base/{STACK}/_index.md`
- Updated: `knowledge-base/{STACK}/README.md`
- Report with trigger keywords and commit instructions
