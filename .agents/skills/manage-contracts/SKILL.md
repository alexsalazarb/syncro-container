---
name: manage-contracts
description: >-
  Manages interface contracts within development plans. Supports creating,
  freezing, updating, and deprecating contracts that define shared interfaces
  between parallel tasks or across projects in a container. Use when parallel
  tasks need to agree on data shapes, API payloads, or behavioral requirements.
---

# Manage Contracts

## Context Required
HIGH-CONTEXT: Plan overview, existing contracts, AGENTS.md

## Triggers

### Automatic
- None (manual only)

### Manual
- `/manage-contracts`
- `/manage-contracts {plan-slug}`
- "create a contract"
- "freeze the contract"
- "update contract"
- "deprecate contract"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` per [`common/config-defaults.md`](../common/config-defaults.md). Extract: `LAYOUT`, `PLANS_DIR`.

### Step 1: Determine Action

Identify the requested operation:
- **Create** — New contract for a plan
- **Freeze** — Lock a draft contract for implementation
- **Update** — Modify a frozen contract (requires change protocol)
- **Deprecate** — Mark a contract as superseded or plan-complete

If not specified, ask the user which action to perform.

### Step 2: Create Contract

1. Gather contract details:
   - Name (descriptive, e.g., "Auth Token Contract")
   - Plan slug (which plan this belongs to)
   - Boundary description (what interface this governs)
   - Producer and consumer (which tasks or projects produce/consume this interface)
   - Scope:
     - **task-level** — contract between parallel tasks within a single plan
     - **cross-project** — contract between projects in a container (e.g. iOS ↔ backend API shape)

2. Resolve the contract path based on scope:
   - **task-level**: `{PLANS_DIR}/{plan-slug}/contracts/`
   - **cross-project** (container layout only):
     - Read the current plan's `overview.md` `Master Plan` field to find the master plan slug
     - If `Master Plan` is absent or `None`, inform the user they must link a master plan first (run `/create-plan --master-plan {slug}` or create a master plan manually)
     - Contract lives at: `{PLANS_DIR}/{master-plan-slug}/contracts/`

3. Select format based on what the contract defines:
   - **TypeScript / Swift / Kotlin interfaces** — Data structures, API payloads, event schemas
   - **Markdown** — Behavioral requirements, protocols, conventions
   - **OpenAPI** — REST API endpoints
   - Formats can be combined in a single contract file

4. Generate contract from `skills/create-plan/references/plan-contract.md` template
5. Place in the resolved path from Step 2
6. Set status to `Draft`
7. Register the contract:
   - **task-level**: add a row to the `Contract References` table in `{PLANS_DIR}/{plan-slug}/overview.md`
   - **cross-project**: add a row to the `Contract References` table in `{PLANS_DIR}/{master-plan-slug}/overview.md`

### Step 3: Freeze Contract

Before freezing, verify:
1. **Open Questions** section is empty (all resolved)
2. All consuming tasks have reviewed the contract
3. Type definitions and behavioral requirements are complete

Then:
1. Update status from `Draft` to `Frozen`
2. Set **Frozen** date
3. Add Change Log entry
4. Update consuming tasks: add a reference to the frozen contract in each consumer task's `task.md` Context section

### Step 4: Update Contract (Change Protocol)

For frozen contracts, all changes require the change protocol:

1. **Identify** — What needs to change and why
2. **Assess impact** — Which consuming tasks/projects are affected
3. **Get approval** — Confirm with stakeholders (report to user)
4. **Update** — Modify the contract, add Change Log entry with date, change description, reason, and approver
5. **Propagate** — Update affected `status.md` files to note the contract change

For draft contracts, changes can be made directly with a Change Log entry.

**Amendment during execution** (triggered by execute-task contract accuracy check):
When an executing task discovers the contract no longer matches reality:
1. The task's `status.md` Adaptations section adds a structured marker:
   `- contract-amendment: {contract-name} | {recommendation} | {task-path}`
2. At the phase gate, `execute-plan` prompts: "Contract {name} needs amendment"
3. Run `/manage-contracts update` with the amendment details
4. After amendment, notify consuming tasks that have not started yet

### Step 5: Deprecate Contract

1. Set status to `Deprecated`
2. Add completion note explaining why (plan complete, superseded, etc.)
3. If superseded, reference the replacement contract
4. Add final Change Log entry
5. Remove or archive from the plan overview's Contract References table

### Step 6: Report

Present the result:
- Contract name and current status
- Location (`{PLANS_DIR}/{plan-slug}/contracts/` or `{PLANS_DIR}/{master-plan-slug}/contracts/`)
- Consuming tasks/projects affected
- Next steps (e.g., "Contract is Draft — review with consumers, then freeze before implementation begins")

## Output

- Contract file created/updated in the resolved contracts path
- Plan overview Contract References table updated
- Change Log entry added for all modifications
- Status report presented to user

## Examples

**task-level contract:**
User: `/manage-contracts create auth-token for user-auth plan`

Agent:
1. Gathers: JWT shape, backend produces, iOS + Android consume
2. Generates contract from template
3. Places at `.plans/user-auth/contracts/auth-token.md`
4. Updates `overview.md` Contract References table
5. Reports: "Created 'Auth Token Contract' (Draft) at `.plans/user-auth/contracts/auth-token.md`. Review with consumers, then run `/manage-contracts freeze` to lock for implementation."

**cross-project contract:**
User: `/manage-contracts create article-api cross-project`

Agent:
1. Reads plan's `overview.md` → `Master Plan: content-platform`
2. Places at `.plans/content-platform/contracts/article-api.md`
3. Reports: "Created 'Article API Contract' (Draft) at master plan level. iOS, Android, and backend tasks can reference this once frozen."
