# {CONTRACT NAME}

<!-- Replace with a clear name describing the interface, e.g., "JWT Claims Contract",
     "SSE Events Contract", "Webhook Payload Schema" -->

**Status**: Draft
**Created**: {YYYY-MM-DD}
**Frozen**: N/A
**Last Updated**: {YYYY-MM-DD}
**Source of Truth**: {architecture doc path, or "this file"}
**Plan**: {plan name} ({plan-slug})
**Scope**: repo-level | cross-repo
**Consumed By**: {Repo/team (role)} -- e.g., "AI Platform (validates), UI (reads)"

<!-- Status values: Draft | Frozen | Deprecated -->
<!-- Draft: open for feedback. Frozen: locked for implementation. Deprecated: superseded or plan complete. -->
<!-- All open questions must be resolved before moving to Frozen. -->

---

## Overview

<!-- 1-3 sentences. What does this contract define? What boundary does it govern?
     A contract describes what one system promises to another at the boundary --
     not how either system works internally. -->
{Describe the interface this contract governs and why it exists.}

---

## Type Definitions

<!-- Use TypeScript interfaces for data structures (canonical even when the producer is
     not TypeScript). Use OpenAPI for REST endpoints. Use both if needed.
     Include inline comments explaining each field. -->

```typescript
/**
 * {Interface description}
 * Producer: {who creates this data}
 * Consumer: {who reads/validates this data}
 */
interface {InterfaceName} {
  field_one: string;      // {description}
  field_two: number;      // {description}
  optional_field?: {       // {description}
    nested: boolean;
  };
}
```

<!-- For OpenAPI contracts, use YAML format:

```yaml
paths:
  /api/v3/{resource}:
    post:
      summary: {description}
      responses:
        '200':
          content:
            application/json:
              schema:
                type: object
                properties:
                  field_one:
                    type: string
```
-->

---

## Behavioral Requirements

<!-- Requirements that type definitions cannot capture: sequencing, error handling,
     retry logic, timing constraints, validation rules. Use numbered lists and
     structured formats -- agents follow these more reliably than prose paragraphs. -->

1. {Behavioral requirement 1}
2. {Behavioral requirement 2}
3. {Error handling requirement}

---

## Examples

<!-- At least one happy-path and one edge-case example. Show the exact JSON/payload
     as it appears on the wire (after serialization). -->

### Happy Path

```json
{
  "field_one": "example_value",
  "field_two": 42
}
```

### Edge Case: {description}

```json
{
  "field_one": "edge_case_value",
  "field_two": 0
}
```

---

## Consuming Repos

| Repo | Role | Key Files |
|------|------|-----------|
| {repo-name} | Producer | {file paths that produce this data} |
| {repo-name} | Consumer | {file paths that consume this data} |

---

## Verification Approach

<!-- How compliance with this contract is verified. CI checks, integration tests,
     manual validation steps. -->

- {e.g., "TypeScript compilation against contract types in CI"}
- {e.g., "Integration test replays example payloads and validates response shape"}
- {e.g., "PR review checklist: verify all required fields present, types match"}

---

## Change Log

| Date | Change | Reason | Approved By |
|------|--------|--------|-------------|
| {YYYY-MM-DD} | Initial draft | -- | -- |

<!-- Append a row for every change. For frozen contracts, all changes require the
     Change Protocol: identify -> notify -> approve -> update -> propagate.
     See the contracts spec for the full protocol. -->

---

## Open Questions

<!-- Unresolved ambiguities. MUST be empty before status moves to Frozen.
     Number each question for easy reference. -->

1. {Question about the interface}
2. {Another open item}

<!-- Delete entries as they are resolved. Record the resolution in the Change Log
     or inline in the relevant section above. -->
