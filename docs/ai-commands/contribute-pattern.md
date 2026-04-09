
# Contribute Pattern

## Context Required
LOW-CONTEXT: AGENTS.md, the pattern to contribute

## Triggers

### Automatic
- After `/document-solution`: if the AI determines the pattern is stack-generic
  (not project-specific), suggest running this skill
- After `/log-mistake`: if the mistake is a pattern any {stack} developer could
  fall into, suggest contributing the correction to the framework

### Manual
- `/contribute-pattern` — contribute one specific pattern
- `/contribute-pattern --scan` — scan the whole project KB for promotable patterns
- "add this to the framework"
- "this should be in the framework KB"
- "other projects should know this"
- "check what can be moved to the framework"
- "scan the KB for reusable patterns"

## Instructions

### Step 0: Resolve context

Follow [`common/resolve-project-context.md`](../common/resolve-project-context.md):

Read `.ai-framework.config` to get `STACK`. Resolve repo root (directory containing `.ai-framework/`).

Resolved variables used throughout this skill:
- **`FRAMEWORK_KB_DIR`** — `.ai-framework/knowledge-base/{STACK}/` (Layer 1 framework KB; the contribution target)
- **`KB_DIR`** — per-project Layer 3 KB root (source for scan mode)
- **`CONTAINER_KB_DIR`** — container-level Layer 2 KB root (also scanned in container layout scan mode)

The framework KB lives at `{FRAMEWORK_KB_DIR}` relative to the repo root — **not** inside `PROJECT_CODE_ROOT` when using container layout.

If `.ai-framework/` is not present (framework not installed as submodule), stop and
tell the user: "Framework submodule not found at .ai-framework/ — cannot contribute directly.
Note the pattern manually and submit it to the framework repo."

### Step 1: Determine mode

**Single mode** (default): User has a specific pattern in mind, or skill was triggered
after `document-solution` / `log-mistake`. Go to Step 2.

**Scan mode** (`--scan` flag, or user says "check what can be moved to the framework"):
Go to Step 1S below, then loop through findings using Steps 2–6.

---

#### Step 1S: Scan the project KB (scan mode only)

Read every `.md` file under `{KB_DIR}/` (and `{CONTAINER_KB_DIR}/` if container layout).
Skip:
- `README.md`
- `best-practices/` (already in framework or project-specific standards)
- `sessions/` (handoff docs — never generic)
- `ai-patterns/mistake-log.md` (project-specific mistakes)

For each doc, evaluate against these questions:
1. **Is the content stack-generic?** — Could it apply word-for-word to a different project using the same stack, without referencing project-specific classes, business logic, or infrastructure?
2. **Does it already exist in the framework?** — Check `.ai-framework/knowledge-base/{STACK}/` for an equivalent doc.
3. **Which tier does it belong to?** — Use the decision table in Step 3.

Classify each doc as one of:
- `PROMOTE` — generic, not yet in framework, clear tier
- `PARTIAL` — partly generic, partly project-specific (needs extraction)
- `SKIP` — project-specific, already in framework, or not applicable

Present findings before doing anything:

```text
KB Scan Results — {PROJECT_DIR} ({STACK})
==========================================

PROMOTE (ready to contribute as-is):
  [1] architecture/coordinator-pattern.md  → framework: architecture/
  [2] integrations/combine-patterns.md     → framework: integrations/

PARTIAL (generic content mixed with project-specific — needs extraction):
  [3] technical/architecture/data-flow.md   → extract "Unidirectional data flow" section
  [4] technical/architecture/threading-issues.md → extract "MainActor migration" section

SKIP (project-specific or already in framework):
  - product/features/article-feed.md       (product feature — project-specific)
  - technical/architecture/app-manager.md  (project class — not generic)

Contribute all PROMOTE items? (y)
Contribute selected items? (list numbers)
Review PARTIAL items one by one? (p)
Cancel? (n)
```

Wait for user confirmation, then process confirmed items through Steps 2–6.

---

### Step 2: Understand the pattern

If invoked in single mode (no prior context), ask the user:
- "What is the pattern or rule you want to contribute?"
- "What problem does it solve?"
- "Is it universal to all {STACK} projects, or does it require a specific library?"

If invoked after `document-solution` or `log-mistake`, use the context already available.

### Step 3: Determine the tier and destination

Use this decision table to pick where the pattern belongs:

| Pattern type | Destination | Example |
|-------------|-------------|---------|
| Rule every {stack} project must follow, regardless of libraries | `knowledge-base/{stack}/best-practices/{stack}-standards.md` (append section) | "Always localize user-facing strings" |
| Architecture/testing pattern reusable in any project | `knowledge-base/{stack}/architecture/{pattern-name}.md` | "Coordinator pattern for navigation" |
| Pattern specific to a library (only useful if the project uses it) | `knowledge-base/{stack}/integrations/{library}-patterns.md` | "How to use Combine for async state" |
| Pattern applicable to ALL stacks | `knowledge-base/generic/best-practices/engineering-standards.md` (append section) | "Never hardcode secrets" |

If the pattern is best-practice tier, append a new section to the existing standards doc
rather than creating a new file. If the file would become unwieldy (>200 lines), split it.

### Step 4: Check for duplicates

Before writing, scan the target file/directory for an existing entry covering the same topic.
If a similar entry exists:
- If the new finding contradicts it → update the existing entry, note the change
- If it adds nuance → add a subsection to the existing entry
- If it's identical → skip, tell the user it's already covered

### Step 5: Write the pattern via `add-kb-doc`

**For best-practice additions** (appending to an existing standards doc):
Write the new section directly into the existing `{stack}-standards.md`. Follow the format
of existing sections in that file. Then update `_index.md` triggers if the new section
introduces keywords not already listed.

**For new architecture/integration docs** (creating a new file):
Delegate to the `add-kb-doc` skill, which handles the full lifecycle:
1. Creates the full document following KB doc structure
2. Generates a `.compact.md` variant if the doc is over 100 lines
3. Updates `_index.md` with trigger keywords and line counts
4. Updates the stack's `README.md` catalog

Run: `/add-kb-doc` with the pattern content gathered in Step 2.

### Step 6: Note what changed

Tell the user exactly what was written and where:

```text
Pattern contributed to framework:
  .ai-framework/knowledge-base/{stack}/{category}/{filename}.md
  → {brief description of what was added}

Tiered loading artifacts:
  _index.md updated with triggers: {keywords}
  .compact.md variant: {created / not needed (< 100 lines) / N/A (appended to existing)}

This will be available to all new {STACK} projects after running:
  .ai-framework/scripts/sync-skills.sh   ← copies skills/agents
  .ai-framework/scripts/init-repo.sh     ← seeds best-practices into new project KBs
```

### Step 7: Suggest committing to the framework

Remind the user that `.ai-framework/` is a submodule and the change lives there:

```text
To share this with everyone, commit it in the framework repo:
  cd .ai-framework
  git add knowledge-base/
  git commit -m "feat({stack}): add {pattern name} to KB"
  git push
```

## Output

- Pattern written to the correct tier in `.ai-framework/knowledge-base/`
- Clear explanation of where it was added and why that tier
- Instructions for committing to the framework repo
