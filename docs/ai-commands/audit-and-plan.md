
# Audit and Plan

## Context Required
LOW-CONTEXT: reads the codebase directly — no prior context needed

## Triggers

### Automatic
- None

### Manual
- `/audit-and-plan`
- "audit the codebase"
- "understand this project"
- "document the architecture"
- "onboard me to this project"

## Instructions

Work through each phase in order. Adapt depth to what you find — a large
codebase may need you to audit one layer at a time.

---

### Step 0: Resolve project context

Follow [`common/resolve-project-context.md`](../common/resolve-project-context.md):

1. Read `.ai-framework.config` → build `folder→stack` map.
2. Determine audit scope:
   - **Single project audit** — one folder was specified, or layout is `single`
   - **Full container audit** — all projects in the `PROJECTS` map (recommended on first onboarding)
   - Ask if unclear: "Audit one project or all projects?"
3. For each project being audited, resolve:
   - **`PROJECT_CODE_ROOT`** — product source tree (`{folder}/` in container layout)
   - **`PROJECT_CONTEXT_ROOT`** — Layer 3 agent context (may differ when `isolated` — see resolve-project-context)
   - **`STACK`** — from map lookup
   - **`KB_DIR`** — per-project Layer 3 root (§4 `resolve-project-context`; equals `PROJECT_CONTEXT_ROOT` when isolated)
   - **`AGENTS_MD`** — `{PROJECT_CONTEXT_ROOT}AGENTS.md`
4. Resolve shared:
   - **`CONTAINER_KB_DIR`** — `docs/kb-container/` at container root (same as `KB_DIR` in single layout)
   - **`PLANS_DIR`** — from config

---

### Phase 1: Classify the codebase

Before diving in, ask the user one question:

```
What state is this codebase in?
  A) Well-structured — built by experienced developers, follows patterns
  B) Partially structured — some good parts, some mess, grew organically
  C) Messy / non-developer — minimal structure, logic accumulated over time
```

This sets the **audit lens**:
- **A** → Focus on documenting patterns, architecture, business logic. Problems section is brief.
- **B** → Document what's good, note where it diverges, flag debt.
- **C** → Assume nothing is intentional. Document what it *actually does*.

Note the classification in the final report. The rest of this skill adapts its tone accordingly.

---

### Phase 2: Map the project

#### 2.1 Understand the surface (stack-specific)

Run the stack-specific discovery commands substituting `{PROJECT_CODE_ROOT}` with the resolved code root. Read `AGENTS_MD` and scan Layer 3 KB under `PROJECT_CONTEXT_ROOT`.

> See [`references/stack-discovery-commands.md`](references/stack-discovery-commands.md) for per-stack commands (Swift, Rails, React, Python, Generic).

#### 2.2 Count and categorize

How many files of the primary type? What are the largest? What directories exist at the top level of `{PROJECT_CODE_ROOT}`?

Read the 5–10 largest files first — they usually contain the most logic.

---

### Phases 3–5: Evaluate architecture, domain, and problems

Follow the structured evaluation checklist across three dimensions:

> See [`references/audit-checklist.md`](references/audit-checklist.md) for:
> - Phase 3: Architecture & pattern questions
> - Phase 4: Business logic & product domain analysis (with KB routing rules)
> - Phase 5: Problem severity classification

Adapt depth based on the Class from Phase 1:
- **Class A** → Document patterns; keep problems brief.
- **Class B** → Balance solid patterns with debt flagging.
- **Class C** → Focus on what the code actually does; problems are primary output.

Write down conventions as you go — they become rules in `{AGENTS_MD}`.

#### Phase 4 output: mandatory written lists

At the end of Phase 4 you MUST produce three written lists — not mental notes.
These feed directly into Phase 6 as mandatory inputs for doc creation.

1. **Domain concepts list** — each concept becomes a `{CONTAINER_KB_DIR}/product/domain/{concept}.md`
   - e.g. `therapy-flow`, `device-lifecycle`, `authentication-flow`, `subscription-model`
2. **Platform features list per project** — each becomes a `{KB_DIR}/product/features/{feature}.md`
   - e.g. ios: `ble-pairing`, `snore-recording`, `therapy-session`; backend: `clinical-portal`, `background-jobs`
3. **Integrations list per project** — each major third-party SDK becomes a `{KB_DIR}/technical/integrations/{integration}.md`
   - e.g. ios: `firebase-integration`; android: `nordic-ble`; backend: `salesforce-sync`

If any list is empty, state that explicitly (e.g. "No platform-only features identified for this project").

---

### Phase 6: Generate KB docs

Phase 6 MUST produce **at minimum** the following docs. Use the Phase 4 written lists
as mandatory inputs — every item on those lists becomes a doc.

**Per project (`{KB_DIR}/`):**

- `technical/architecture/project-overview.md` — stack, entry points, structure
- `technical/architecture/architecture-patterns.md` — architecture style, DI, data flow, navigation
- `technical/architecture/coding-conventions.md` — file org, naming, async, error handling
- `technical/integrations/{name}.md` — **one per major SDK** from the Phase 4 integrations list
- `product/features/{name}.md` — **one per feature** from the Phase 4 platform features list
- `ai-patterns/known-issues.md` — problems found (Class B/C; brief for Class A)

**Container shared (`{CONTAINER_KB_DIR}/`):**

- `product/domain/product-overview.md` — what the product does, platform matrix, glossary
- `product/domain/{concept}.md` — **one per concept** from the Phase 4 domain concepts list
- `technical/api/{api}.md` — one per shared REST API consumed by multiple stacks
- `technical/ble/{spec}.md` — if BLE or hardware protocol exists
- `technical/data-models/{model}.md` — if canonical data shapes are shared across platforms

> See [`references/kb-generation-rules.md`](references/kb-generation-rules.md) for:
> - Detailed KB routing rules (which layer, which category)
> - KB doc template
> - Three-tier copy rules (framework → project)

---

### Phase 6b: Detect framework KB gaps

During Phase 2, you mapped the project's dependencies. Now cross-reference them against
the framework KB at `{FRAMEWORK_KB_DIR}/`:

1. List all `.md` files and `_index.md` trigger keywords in `{FRAMEWORK_KB_DIR}/`
2. For each library/SDK identified in Phase 2 dependency scanning, check if a matching
   KB doc exists (by filename or trigger keyword)
3. Exclude core-platform basics (Foundation, UIKit, stdlib, etc.)

If gaps are found, include them in the final report and suggest:

```text
Framework KB gaps detected — {N} technologies used in this project have no KB doc:
  - {tech1}, {tech2}, ...

Run `/research-kb-topic --scan` to research and populate these docs.
```

Do NOT auto-run the research — the audit is already a large operation. Flag the gaps
and let the user decide when to fill them.

---

### Phase 6c: Verify output completeness

Before proceeding to Phase 7, verify that Phase 6 produced the full set of docs.
Walk through each list from Phase 4 and confirm a matching file exists.

1. **Domain concepts**: cross-reference the Phase 4a domain concepts list — does each
   have a doc in `{CONTAINER_KB_DIR}/product/domain/`?
2. **Platform features**: cross-reference the Phase 4b platform features list per project —
   does each project have docs in `{KB_DIR}/product/features/`?
3. **Integrations**: cross-reference the integrations list — does each project have docs
   in `{KB_DIR}/technical/integrations/` for its major third-party SDKs?
4. **Container audit symmetry** (container layout only): if a shared feature
   (e.g. BLE pairing, therapy session) has a `product/features/` doc on one project,
   check whether other projects that implement the same feature also need one.
   Each platform with meaningful differences (permissions, libraries, UI patterns) gets its own doc.

If any are missing, create them before continuing.

---

### Phase 7: Update AGENTS.md

Update **both** the per-project and container-level AGENTS.md files.

#### 7a. Container-level `{CONTAINER_AGENTS_MD}` (container layout only)

In container layouts, the root `AGENTS.md` describes the **overall product and workspace** — not any single stack. Look for `<!-- CUSTOMIZE -->` markers and fill them in.

**What to add / update:**
- **Project Context** — product name, one-sentence description, what the product does
- **Aspect table** — list each sub-project with its stack. Include main branch, deployment info, shared testing strategy.
- **Key commands** — container-level commands that apply across projects. Per-project commands go in `{AGENTS_MD}`.
- **Critical Constraints** — constraints that apply across the whole product
- **KB references** — confirm the 3-Layer KB loading paths are correct for this container

Skip sections that only make sense at the per-project level (architecture patterns, key files, naming conventions — those belong in `{AGENTS_MD}`).

#### 7b. Per-project `{AGENTS_MD}`

For each project audited, update its `{AGENTS_MD}` with what you learned.

Look for `<!-- CUSTOMIZE -->` markers and fill them in. If already customized, add or update:
- **Stack & Architecture**: Confirm the actual stack, architecture pattern, key libraries
- **Key Files**: Entry points, base classes, route files, service registries, etc.
- **Patterns to Follow**: Conventions discovered in Phases 3–5
- **Patterns to Avoid**: Anti-patterns found in the audit
- **KB Index**: Confirm the KB docs created in Phase 6 are listed

Do NOT overwrite content the user has already customized. Add to it.

---

### Phase 8: Create improvement plan (optional)

Ask the user:
```
Do you want me to create an improvement plan based on the audit findings?
(Recommended for Class B and C codebases, or if 🔴/🟠 issues were found)
```

If yes, run `/create-plan` with full context of findings, problems sorted by severity,
files involved, and suggested approach per fix.

Plan naming: `{project-name}-codebase-improvement`

For Class A codebases, the plan may be short (a few 🟡 items) or skipped entirely.

---

### Phase 9: Update KB index and report

Run `/check-kb-index` for each `{KB_DIR}` updated, and for `{CONTAINER_KB_DIR}` if shared docs were written.

> See [`references/improvement-plan-template.md`](references/improvement-plan-template.md) for the final report format.

## Output

- Per-project KB docs in `{KB_DIR}/` — `technical/architecture/`, `engineering/best-practices/`, `technical/integrations/`, etc.
- Shared KB docs in `{CONTAINER_KB_DIR}/` — `product/domain/`, `technical/api/`, `technical/data-models/`, `engineering/`
- Updated `{AGENTS_MD}` per project with discovered conventions
- Updated KB `README.md` index per location touched
- (Optional) Improvement plan in `{PLANS_DIR}/`
