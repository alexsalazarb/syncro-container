
# Cleanup KB

## Context Required
LOW-CONTEXT: AGENTS.md, dependency files

## Triggers

### Automatic
- None (manual only)

### Manual
- `/cleanup-kb`
- "clean up the knowledge base"
- "remove unused KB docs"
- "KB has stale docs"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve `STACK`, `PROJECT_CODE_ROOT`, `PROJECT_CONTEXT_ROOT`, `LAYOUT`, and `PROJECTS` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).

In container layout, ask the user which project to clean (or "all"). Resolve `KB_DIR` for
each project in scope plus `CONTAINER_KB_DIR` for the container root.

### Step 1: Scan dependency files

Read the project's dependency manifest under **`PROJECT_CODE_ROOT`** (product tree, not the isolated agent overlay) to build a list of libraries actually in use:

| Stack | File(s) to read |
|-------|----------------|
| Swift | `Podfile`, `Package.swift` |
| Rails | `Gemfile` |
| React | `package.json` (dependencies + devDependencies) |
| Python | `requirements.txt`, `pyproject.toml`, `Pipfile` |
| Generic | Any `*file`, `*.toml`, `*.json` with dependencies |

Extract library/package names into a flat list.

### Step 2: Scan KB docs

List all `.md` files under `{KB_DIR}/` (and `{CONTAINER_KB_DIR}/` if container).
Exclude:
- `README.md` (index file — never delete)
- `engineering/best-practices/` (standards — never delete)
- `ai-patterns/mistake-log.md` (always relevant)
- `ai-patterns/sessions/` (handoff docs — never delete)
- Anything the user explicitly created (heuristic: has substantial content, >20 lines)

> **Container KB structure** (`CONTAINER_KB_DIR`): the new layout uses `product/domain/`,
> `technical/api/`, `technical/ble/`, `technical/data-models/`, `engineering/best-practices/`,
> `engineering/security/`, `engineering/testing/`, and `ai-patterns/sessions/`.
> Scan these paths when cleaning the container KB.

### Step 3: Identify candidates for removal

For each KB doc, determine if it should be removed. There are three reasons to remove:

**a. Library doc for an unused library**
The doc name or content references a specific library not found in Step 1.

Common patterns:
- `rxswift-patterns.md` → requires `RxSwift` in Podfile/Package.swift
- `realm-patterns.md` → requires `RealmSwift` or `Realm` 
- `sidekiq-patterns.md` → requires `sidekiq` gem
- `celery-patterns.md` → requires `celery` Python package
- `redux-patterns.md` → requires `redux` or `@reduxjs/toolkit` in package.json

**b. Empty stub (seeded but never populated)**
The doc exists, has fewer than 10 lines of real content, and contains only placeholder
text (e.g. `*No documents yet*`, `TODO`, template markers).

**c. Orphaned index entry**
The `README.md` index references a file that no longer exists on disk.

### Step 4: Present findings and ask for confirmation

Show a grouped summary before taking any action:

```text
KB Cleanup — {PROJECT_DIR} ({STACK})
=====================================

Library docs for unused libraries:
  [ ] technical/integrations/rxswift-patterns.md  (RxSwift not in Podfile)
  [ ] technical/integrations/realm-patterns.md    (RealmSwift not in Podfile)

Empty stubs (never populated):
  [ ] technical/architecture/di-patterns.md       (8 lines, all placeholder text)

Orphaned index entries in README.md:
  [ ] product/features/cross-repo-plans.md        (file does not exist)

3 items found.

Remove all? (y), remove selected (list numbers), or cancel (n)?
```

Wait for user confirmation before proceeding.

### Step 5: Remove confirmed items

For each confirmed item:
- Delete the file from disk
- Note it for README index cleanup in Step 6

For orphaned index entries: only remove the README row, no file to delete.

### Step 6: Update KB index

Run `/check-kb-index` to rebuild the `README.md` index after deletions.

### Step 7: Report

```text
KB Cleanup Complete
===================
Removed: 3 files
  ✓ integrations/rxswift-patterns.md
  ✓ integrations/realm-patterns.md
  ✓ architecture/di-patterns.md

KB index updated.
```

## Output

- List of removed files
- Updated KB README index
- Clean KB with only relevant documentation
