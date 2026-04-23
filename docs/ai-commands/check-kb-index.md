
# Check KB Index

## Context Required
LOW-CONTEXT: AGENTS.md only

## Triggers

### Automatic
- After creating, modifying, or deleting KB markdown under `{CONTAINER_KB_DIR}` (Layer 2 — default `docs/kb-container/`) or `{KB_DIR}` (Layer 3 — e.g. `docs/kb-projects/<project>/` when isolated)
- After running `document-solution` skill

**Exclude**: Do not retrigger when the only change is a `README.md` in the affected KB root that `check-kb-index` itself just rewrote.

### Manual
- `/check-kb-index`
- "run check-kb-index"
- "update kb index"
- "sync knowledge base"

## Instructions

### Step 0: Resolve project context

Follow the algorithm in [`common/resolve-project-context.md`](../common/resolve-project-context.md):

1. Read `.ai-framework.config` → build `folder:stack` map from `PROJECTS` (or `PROJECT_DIR`/`STACK` for single layout).
2. Infer the active project from the file that was just created/modified (its path prefix identifies the project). If ambiguous, ask.
3. Resolve **`KB_DIR`** and **`CONTAINER_KB_DIR`** per §4 of [`common/resolve-project-context.md`](../common/resolve-project-context.md). If the changed file lives under `CONTAINER_KB_DIR`, update that root's `README.md`; if under `KB_DIR`, update the project KB `README.md`.

> Use the correct root for the path that changed. Project folder keys are arbitrary — do not assume they match the stack name.

### Step 1: Determine which root changed

If the changed file lives under `{CONTAINER_KB_DIR}`, set `TARGET_ROOT={CONTAINER_KB_DIR}`.
If the changed file lives under `{KB_DIR}`, set `TARGET_ROOT={KB_DIR}`.
In single layout these are the same path — use `{CONTAINER_KB_DIR}`.

Scan all KB files under `TARGET_ROOT`:
```bash
find {TARGET_ROOT} -name "*.md" -not -name "README.md"
```

### Step 1a: YAML validation (when `kb-index.yaml` exists)

> **Note:** v1 does not auto-regenerate README from YAML. That remains future work (deferred from the `l2-l3-kb-index-yaml` plan). `generate-kb-index` goes the other direction: it generates a `kb-index.yaml` stub from an existing README-based KB.

Check whether `{TARGET_ROOT}/kb-index.yaml` exists.

**If `kb-index.yaml` is absent** — skip this step entirely. Behavior is unchanged (README-only flow, Steps 2–7 below).

**If `kb-index.yaml` is present:**

1. **Validate the YAML** — run the validation script:
   ```bash
   {FRAMEWORK_ROOT}/scripts/validate-kb-index-yaml.sh {TARGET_ROOT}
   ```
   Report any errors or warnings produced. If the script exits non-zero, surface the full output and stop (do not proceed to README update).

2. **Collect all `.md` files** under `{TARGET_ROOT}`, excluding `README.md`:
   ```bash
   find {TARGET_ROOT} -name "*.md" -not -name "README.md"
   ```

3. **Collect all `path:` values** from `kb-index.yaml`. Each entry in the `docs` list has a `path:` key; normalize paths to be relative to `{TARGET_ROOT}` for comparison.

4. **Detect orphans** — any `.md` file found in Step 2 that is NOT listed as a `path:` value in Step 3 is an orphan. Warn for each:
   ```text
   WARNING: orphaned KB file not listed in kb-index.yaml:
     technical/architecture/some-undocumented.md
   ```

5. Continue to Steps 2–7 (README update flow) as normal.

### Step 2: Extract metadata for each file
- Category (from directory — same folder names for both Layer 2 and Layer 3):
  - `product/domain/`, `product/features/`
  - `technical/architecture/`, `technical/api/`, `technical/ble/`, `technical/data-models/`, `technical/integrations/`
  - `engineering/best-practices/`, `engineering/security/`, `engineering/testing/`
  - `ai-patterns/`, `ai-patterns/sessions/`
- Filename (without extension)
- "Context" line (from file header): "Read when..."
- Last modified date (from git or file system)

### Step 3: Load current README.md index
- Parse existing index table(s) at `{TARGET_ROOT}/README.md`
- Note current entries

### Step 4: Compare and identify changes
- **New files**: In file system but not in index
- **Removed files**: In index but not in file system
- **Stale dates**: Index date doesn't match file's last modified

### Step 5: Generate updated index sections

For each category, create/update table:
```markdown
### Technical / Architecture
| Document | When to Read | Last Updated |
|----------|--------------|--------------|
| [doc-name](technical/architecture/doc-name.md) | Context trigger | Mon YYYY |
```

### Step 6: Update README.md
- Preserve non-index content (overview, how-to-use sections)
- Replace index tables with updated versions
- Add new category sections if needed
- Remove empty category sections

### Step 7: Report changes
```text
Knowledge Base Index Updated
============================

Added:
+ technical/architecture/some-pattern.md

Removed:
- (none)

Index now has N documents across M categories.
```

## Output

- Updated `{TARGET_ROOT}/README.md` (whichever root was affected — `{CONTAINER_KB_DIR}` or `{KB_DIR}`)
- Console report of changes made
