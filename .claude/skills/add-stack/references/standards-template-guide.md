# Standards Template Guide

Used by [add-stack](../SKILL.md) Step 5 to create `best-practices/{slug}-standards.md`.

---

## Purpose

The standards doc contains **universal, non-negotiable rules** for every project of this
stack. These are always loaded by agents (via `_index.md` "Always Load" section).

Keep it under 120 lines. Rules only — no tutorials or lengthy explanations.

---

## Required sections

Every standards doc MUST cover these areas (adapt naming to the stack):

### 1. Localization
- "All user-facing strings must be localized."
- Show a wrong/right example in the stack's language.
- Mention that the specific i18n library varies per project.

### 2. Type Safety / Code Safety
- Strictness rules for the language (e.g. no `any` in TS, no force unwraps in Swift,
  no `dynamic` in Dart, strict mode in Go).
- Lint suppression policy.

### 3. Component / Module Architecture
- One class/component per file rule.
- Naming convention for files.
- Separation of concerns (no business logic in view layer).

### 4. Async / Reactive Patterns
- Error handling requirements for async code.
- Subscription/resource cleanup rules.
- Thread/isolate safety rules.

### 5. Security
- No secrets in source code.
- Sanitization defaults (e.g. XSS prevention, SQL injection).
- Secure storage for sensitive data.

### 6. Testing
- What must be tested (business logic, components, integration points).
- Testing framework to use (or "per project convention").
- Rule: no production changes without test updates.

---

## Format

Follow the structure of `knowledge-base/flutter/best-practices/flutter-standards.md`:

```markdown
# {Stack} Project Standards

Universal quality standards that apply to every {stack} project regardless of
architecture or libraries used. These are non-negotiable defaults — document
exceptions in the project's `AGENTS.md`.

---

## {Section}

**Rule in bold.** Brief explanation.

\`\`\`{language}
// Wrong
...

// Right
...
\`\`\`

---
```

Use code examples from the stack's language. Show wrong/right pairs where the
distinction is non-obvious.
