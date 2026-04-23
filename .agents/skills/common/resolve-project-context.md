# Resolve Project Context

Reusable Step 0 logic for any skill that needs to know which project it is operating in
and where that project's resources (KB, AGENTS.md, plans) live.

Include in a skill as **Step 0** by writing: `> See [resolve-project-context](../common/resolve-project-context.md)`
then copy the algorithm below directly into the step.

Full contract: [project-ai-context-modes contract](../../.plans/project-ai-context-modes/contracts/ai-context-resolution.md) (embedded vs isolated; path relative to framework repo / `.ai-framework/` submodule).

---

## Algorithm

### 1. Read `.ai-framework.config`

Look for `.ai-framework.config` at the repository root (one level above `.ai-framework/`).

Key fields:
| Field | Description |
|-------|-------------|
| `LAYOUT` | `single` or `container` |
| `PROJECTS` | Comma-separated entries (container only). See grammar below. |
| `PROJECT_DIR` | Single project folder (single layout only). Defaults to `.` |
| `STACK` | Tech stack for single layout |
| `PLANS_DIR` | Path to plans directory. Defaults to `.plans` |
| `CONTAINER_KB_ROOT` | Directory under repo root for **Layer 2** (container-wide KB). Default `docs/kb-container`. Written by `init-repo.sh`; override if you relocate shared KB. |
| `AI_CONTEXT_ROOT` | (Container, optional) Parent directory for **Layer 3** when projects are **isolated** (`AGENTS.md` + KB live under `{AI_CONTEXT_ROOT}/{folder}/`). Default `docs/kb-projects`. Ignored if all projects are embedded. |
| `WORK_PLANS_PATH` | (Optional) Absolute path to an external work-plans hub directory. Used by hub-resolution (tier 1). If unset, falls back to git global config then container self-hosting. See [`common/hub-resolution.md`](hub-resolution.md). |
| `REPOS` | Comma-separated repo entries: `key\|remote-url\|local-path`. Each entry registers a client repo for cloning via `clone-repos.sh`. Each repo lives **inside** the container as a subfolder. Pipe (`\|`) is the field separator because SSH URLs contain colons. Example: `ios\|git@github.com:org/ios.git\|ios` |

**Spotting layers from paths**: `docs/kb-container/` = Layer 2 only. `docs/kb-projects/<project>/` = Layer 3 (isolated). Embedded projects use `<project>/docs/kb-project/` for Layer 3 KB.

If config is absent, assume: `LAYOUT=single`, `PROJECT_DIR=.`, `PLANS_DIR=.plans`, `CONTAINER_KB_ROOT=docs/kb-container`.

**Parse `REPOS`**: Split on `,` to get entries; split each entry on `|` to get `repo_key`, `repo_url`, `repo_local_path`. Build a `repo_key → repo_local_path` lookup table. A project whose `folder` key appears in this table is **registered in REPOS for cloning purposes** — the `repo_local_path` is a subfolder inside the container (relative to container root).

### 1a. Parse each `PROJECTS` entry (container only)

Each entry has the form `folder:stack` or `folder:stack:mode`.

| Colons in entry | `folder` | `stack` | `mode` |
|-----------------|----------|---------|--------|
| 1 | before first `:` | after first `:` | `embedded` (default) |
| 2 | before first `:` | between first and second `:` | third segment — must be `embedded` or `isolated` |

The folder key is arbitrary (e.g. `ios`, `clients/ios`) and must not contain `:`.

Examples:
- `api:rails` → embedded Rails project in `api/`
- `ios:swift:isolated` → Swift code in `ios/`, agent files under `{AI_CONTEXT_ROOT}/ios/`

### 2. Build the folder → (stack, mode) map

```
LAYOUT=single  →  single entry: PROJECT_DIR → (STACK, embedded)
LAYOUT=container  →  parse PROJECTS CSV into map:
    "myapp:swift,backend:rails:isolated"  →  myapp → (swift, embedded), backend → (rails, isolated)
```

The folder name is arbitrary — never assume it matches the stack name.

### 3. Infer the active project from context

Use whichever signal is available (in priority order):

1. **Explicit path passed by caller** — if the triggering event includes a file path
   (e.g. `ios/SomeView.swift`, `backend/app/models/user.rb`), extract the leading
   path segment **that matches a map key** (longest prefix match if keys are nested paths).

2. **Layer 3 isolated paths** — if the user works under `docs/kb-projects/<folder>/...`, infer project `<folder>` from the segment after `AI_CONTEXT_ROOT`.

3. **Files open / recently edited in the IDE** — check visible file paths. Find which
   map key is a path prefix of any of those paths (code under `folder/` or context under `{AI_CONTEXT_ROOT}/{folder}/`).

4. **Caller stated the project** — if the user mentioned a project name in their message,
   match it against the map keys (exact or partial match).

5. **Unambiguous single project** — if the map has exactly one entry, use it without asking.

6. **Cannot infer → ask the user**:
   ```
   Which project does this apply to?
   Options: myapp (swift) | backend (rails)
   ```

### 4. Resolve paths

Let `folder` be the active project key, `stack` and `mode` from the map.
Let `CONTAINER_KB_ROOT` be from config; if unset, treat as `docs/kb-container`.
Let `AI_CONTEXT_ROOT` be from config; if any project uses `isolated` and it is unset, treat as `docs/kb-projects`.

| Variable | Single layout | Container + embedded | Container + isolated |
|----------|--------------|----------------------|----------------------|
| `PROJECT_CODE_ROOT` | `{PROJECT_DIR}/` (or repo root if `.`) | `{folder}/` | `{folder}/` |
| `PROJECT_CONTEXT_ROOT` | same as code root | `{folder}/` | `{AI_CONTEXT_ROOT}/{folder}/` |
| `FRAMEWORK_KB_DIR` | `.ai-framework/knowledge-base/{STACK}` | `.ai-framework/knowledge-base/{stack}` | same |
| `CONTAINER_KB_DIR` | `{CONTAINER_KB_ROOT}/` | `{CONTAINER_KB_ROOT}/` (repo root; Layer 2 only) | same |
| `KB_DIR` (Layer 3) | same as `CONTAINER_KB_DIR` | `{folder}/docs/kb-project/` | `{PROJECT_CONTEXT_ROOT}/` |
| `CONTAINER_AGENTS_MD` | `AGENTS.md` | `AGENTS.md` (repo root) | same |
| `AGENTS_MD` (per-project) | `AGENTS.md` (repo root) | `{PROJECT_CONTEXT_ROOT}AGENTS.md` | same formula |
| `STACK` | from config | from map | from map |
| `PLANS_DIR` | from config | from config (shared at container root) | same |

> In **single** layout, Layer 2 and Layer 3 share one physical tree under `CONTAINER_KB_DIR`. In **container + isolated**, Layer 3 files live entirely under `PROJECT_CONTEXT_ROOT` (same as `KB_DIR`). In **embedded**, `AGENTS.md` sits in `{folder}/` while markdown KB lives under `{folder}/docs/kb-project/`.

**Legacy alias:** Skills written before this split may say `PROJECT_ROOT` when meaning “where application source lives.” Treat **`PROJECT_ROOT` ≡ `PROJECT_CODE_ROOT`** for audits, `find`, implementation steps, and git operations on product code. Do **not** use `PROJECT_ROOT` for Layer 3 `AGENTS.md` or per-project KB paths — use `PROJECT_CONTEXT_ROOT`.

These KB variables form a **3-layer hierarchy** — ordered from broadest to narrowest scope:

| Layer | Variable | Scope | Contains |
|-------|----------|-------|----------|
| **Layer 1** | `FRAMEWORK_KB_DIR` | All projects of a stack, everywhere | Technology patterns. Has `_index.md` for tiered loading. |
| **Layer 2** | `CONTAINER_KB_DIR` | All projects within this repo/product | Product domain, shared API, cross-project features. |
| **Layer 3** | `KB_DIR` | One logical project (agent context) | Project-specific docs. Physical root is `KB_DIR` (§4 — equals `PROJECT_CONTEXT_ROOT` when isolated, not when embedded). |

> `FRAMEWORK_KB_DIR` is always `.ai-framework/knowledge-base/{STACK}` — deterministic from stack.
>
> For **reading** KB: agents check Layer 1 first (`_index.md` for tiered loading), then Layer 2/3 (`README.md` indexes).
> For **writing** KB: skills route new docs to the correct layer based on scope (see Section 5).

**Optional machine index**: `kb-index.yaml` may exist at `CONTAINER_KB_DIR` and `KB_DIR` roots. When present, agents use trigger-based loading (compact-first) instead of README scanning. See `docs/KNOWLEDGE-LAYERING.md` (§ "Optional Machine-Readable Index") and `docs/AGENT-RUNTIME-FLOW.md` for the loading algorithm. Resolution variables are unchanged.

### 4b. Topic standards — multiple Layer 1 approaches *(optional — skip if no multi-approach topics)*

Some concerns (concurrency, persistence, navigation, state) have **several valid technologies** in Layer 1. The framework may ship reference docs for more than one approach.

- **Projects are not required** to document a choice at init time.
- **When a team is ready**, add optional Layer 3 docs under `KB_DIR` — for example `architecture/concurrency-standard.md`, `architecture/persistence-standard.md`, or one combined `architecture/technology-choices.md`. Alternatively document choices in `AGENTS.md` (see `docs/KB_TOPIC_STANDARDS.md` in this repo).
- **Agent load order** for those topics: if such a doc exists, read it **before** loading competing Layer 1 guides; then load only Layer 1 files that match *standard*, *legacy*, or the explicit task (e.g. migration). If none exists, use `_index.md` triggers and prefer compact docs; do not load every alternative deep doc by default.

Full pattern: [KB_TOPIC_STANDARDS.md](../../docs/KB_TOPIC_STANDARDS.md) (from framework repo root; in a consumer repo: `.ai-framework/docs/KB_TOPIC_STANDARDS.md`).

### 5. KB write routing

When writing a KB document, first determine the **layer**, then the **category**:

**Layer decision:**
- "Is this true for ALL projects of this stack?" → Layer 1 (`FRAMEWORK_KB_DIR`) via `add-kb-doc`
- "Is this shared across projects in this repo?" → Layer 2 (`CONTAINER_KB_DIR`) via `document-solution`
- "Is this specific to this one codebase / stack instance?" → Layer 3 (`KB_DIR`) via `document-solution` — resolve `KB_DIR` from the table in §4 (not always equal to `PROJECT_CONTEXT_ROOT` when embedded).

**Category routing — same folder names for Layer 2 and Layer 3** (mirror `product/`, `technical/`, `engineering/`, `ai-patterns/`):

| Subpath | Layer 2 (`CONTAINER_KB_DIR`) | Layer 3 (`KB_DIR`) |
|---------|------------------------------|---------------------|
| `product/domain/` | Shared business rules, clinical/domain concepts | Rare: platform-only domain notes |
| `product/features/` | — (prefer Layer 3) | Screens/capabilities that exist only on this stack |
| `technical/architecture/` | — (prefer Layer 3) | MVVM, navigation, app structure for this codebase |
| `technical/api/` | REST contracts used by multiple stacks | How *this* stack consumes APIs |
| `technical/ble/` | BLE protocol shared across platforms | Platform-specific BLE implementation notes |
| `technical/data-models/` | Canonical shapes across stacks | Stack-specific DTO/view-model notes |
| `technical/integrations/` | — | Third-party SDKs (StoreKit, Play Billing, etc.) |
| `engineering/best-practices/` | Cross-stack conventions | Stack standards (Swift/Kotlin/Python) |
| `engineering/security/` | Org-wide security rules | Platform-specific security notes |
| `engineering/testing/` | Cross-stack test strategy | Stack test patterns |
| `ai-patterns/` | Container mistake log (optional) | Per-project mistake log, known issues |
| `ai-patterns/sessions/` | Session handoffs (`/save-session`) | Optional per-project session notes |

#### Routing in detail

**Shared across stacks** → always `CONTAINER_KB_DIR/...` (Layer 2).

**Specific to one codebase** → `KB_DIR/...` (Layer 3) using the **same** subpaths as above.

**Platform-only features** → `KB_DIR/product/features/` (e.g. `{AI_CONTEXT_ROOT}/{folder}/product/features/` when isolated).

**When a domain feature has meaningful platform differences**, keep the domain doc at Layer 2 and link to the relevant `KB_DIR/technical/` or `KB_DIR/product/features/` note.

> **Decision rules**:
> - "Would an Android dev care?" → yes → Layer 2 (`CONTAINER_KB_DIR`)
> - "Is this about *what* the product does?" → Layer 2 `product/domain/` (or Layer 3 `product/features/` if platform-only UI)
> - "Is this about *how* platforms talk to each other?" → Layer 2 `technical/{api|ble|data-models}/`
> - "Is this about *how we build* things?" → `engineering/` at the layer that matches scope (2 vs 3)
> - "How does *this* codebase implement it?" → Layer 3 `KB_DIR` with the matching category

#### Examples (wording)

Prefer neutral phrasing in skill text:

- "Layer 3 / per-project KB (`KB_DIR`)" instead of only "`{folder}/`".
- "Read per-project `AGENTS.md` at `AGENTS_MD` (`PROJECT_CONTEXT_ROOT`)" instead of assuming `{folder}/AGENTS.md`.

---

### 6. Container repo Git rules (MANDATORY)

These rules apply whenever `LAYOUT=container`. Every skill that touches git MUST follow them.

#### 6a. Never branch the container repo

**The container repo MUST NOT receive task branches or integration branches.** All `git switch -c`, `git checkout -b`, and branch-creating operations run inside `PROJECT_CODE_ROOT` — which is a project subfolder inside the container.

If a skill resolves `PROJECT_CODE_ROOT` to the container root itself (`.` in container layout), that is a misconfiguration — **stop, do not branch, and ask the user to clarify the target project.**

#### 6b. What may auto-commit vs. what requires human approval

| Operation | Examples | Auto-commit? |
|-----------|----------|-------------|
| Plan metadata | `{PLANS_DIR}/*/overview.md`, `status.md`, `README.md` | **Yes** |
| Config / skill sync | `.ai-framework.config`, skill/agent sync via upgrade | **Yes** |
| Layer 2 KB | `{CONTAINER_KB_DIR}/**/*.md` | **No — human approval required (§6c)** |
| Layer 3 KB | `{KB_DIR}/**/*.md` | **No — human approval required (§6c)** |

All application code, tests, and migrations belong in `PROJECT_CODE_ROOT` — never in the container.

#### 6c. All KB additions require human approval — separate PR

This applies to **both Layer 2 (`CONTAINER_KB_DIR`) and Layer 3 (`KB_DIR`)**.

When a skill writes a new or updated file under any KB directory, it MUST:

1. Write the file(s).
2. Run `check-kb-index` to update the index.
3. Show the written content and path to the user.
4. Tell the user to create a **dedicated KB PR** — never mix KB changes with plan files or application code:

```
KB addition ready for review.

  File: {path}
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

**Why both layers need a PR:**
- **Layer 2 (shared)**: product domain docs and technical contracts affect all projects — the whole team needs to review before they land on main.
- **Layer 3 (project-specific)**: architecture and product feature docs are stack-specific — the relevant team needs to validate accuracy.
- **`ai-patterns/` (mistake log, sessions)**: leave written, uncommitted. The dev decides when and whether to push — no PR required, but no auto-commit either.
- **Plans are the only exception**: plan metadata is operational state that changes constantly. Push directly to main, no PR.
