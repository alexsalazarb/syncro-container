---
name: generate-kb-index
description: >-
  Generates a kb-index.yaml stub for KB directories that have a README.md but
  no kb-index.yaml. Wraps scripts/generate-kb-index.sh with project context
  resolution, user confirmation, preview, and validation. Use when bootstrapping
  a new KB layer or migrating an existing README-based KB to YAML indexing.
---

# Generate KB Index

## Context Required
MEDIUM-CONTEXT: .ai-framework.config, KB directory README.md

## Triggers

### Automatic
- None (manual only)

### Manual
- `/generate-kb-index`
- `/generate-kb-index [layer-root]`
- "generate kb index"
- "migrate kb to yaml"
- "create kb-index.yaml"

## Instructions

### Step 0: Resolve project context

Follow the algorithm in [`common/resolve-project-context.md`](../common/resolve-project-context.md):

1. Read `.ai-framework.config` → extract `LAYOUT`, `CONTAINER_KB_ROOT` (default `docs/kb-container`), `AI_CONTEXT_ROOT` (default `docs/kb-projects`), and `PROJECTS` (container layout only).
2. Resolve `CONTAINER_KB_DIR` = `{CONTAINER_KB_ROOT}/`.

---

### Step 1: Identify target KB directories

**If a `[layer-root]` argument was provided**: use it directly as the sole target. Skip the scan.

**Otherwise**, scan for KB directories that have `README.md` but no `kb-index.yaml`:

1. Check `CONTAINER_KB_DIR` (Layer 2 — container):
   - If `{CONTAINER_KB_DIR}/README.md` exists and `{CONTAINER_KB_DIR}/kb-index.yaml` does NOT exist → add to list
   - If `kb-index.yaml` already exists → note "already indexed"

2. For container layout, check each project's Layer 3 KB root:
   - Embedded: `{folder}/docs/kb-project/`
   - Isolated: `{AI_CONTEXT_ROOT}/{folder}/`
   - Apply same README.md / kb-index.yaml check

If no eligible directories found:
```
No KB directories found that need a kb-index.yaml.
All KB roots already have kb-index.yaml, or no README-based KB was found.
```
Exit.

---

### Step 2: Confirm with user

List eligible directories and ask which to generate for:

```
Found KB directories without kb-index.yaml:
  1. docs/kb-container/  (Layer 2 — container)
  2. ios/docs/kb-project/ (Layer 3 — ios)

Generate kb-index.yaml for: [all / 1 / 2 / comma-separated numbers]
```

Accept "all" or individual numbers.

---

### Step 3: Run the generator script

For each confirmed directory, run:
```bash
.ai-framework/scripts/generate-kb-index.sh <layer-root> [--layer container|project]
```

Pass `--layer container` for Layer 2 directories; `--layer project` for Layer 3.

Display the script output as it runs.

---

### Step 4: Show preview

Display the generated `kb-index.yaml` content:
```
Preview: docs/kb-container/kb-index.yaml
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[first 40 lines of generated file]
...
[N more lines — open file to see all]
```

---

### Step 5: Run the validator

For each generated file, run:
```bash
.ai-framework/scripts/validate-kb-index-yaml.sh <layer-root>
```

Display the validation output. If errors appear, explain them and stop — do not proceed to Step 6 until the user resolves them.

---

### Step 6: Remind user to review triggers

```
Next steps:
  1. Open the generated kb-index.yaml and review each document's `triggers` list.
     — Add domain terms developers would naturally write (e.g. "feature flag", "A/B test")
     — Remove generic words that don't help route the document ("overview", "notes")
  2. Add `load_with:` for closely related documents that should be read together.
  3. Add `compact:` paths if compact variants exist or will be created.
  4. Commit in a dedicated KB PR:
       git checkout -b kb/index-{layer}
       git add <layer-root>/kb-index.yaml
       git commit -m "docs(kb): add kb-index.yaml for {layer}"
       git push -u origin kb/index-{layer}
     Then open a PR targeting main. Keep KB PRs separate from plan files.
```

## Output

- `kb-index.yaml` stub generated for each confirmed layer root
- Validation result shown
- User guided to review triggers before committing
