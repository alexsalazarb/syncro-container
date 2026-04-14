# AGENTS Template Guide

Used by [add-stack](../SKILL.md) Step 4 to create `templates/AGENTS.{slug}.md`.

---

## Structural base

Copy the section order and boilerplate from an existing template (e.g. `templates/AGENTS.react.md`).
The following sections are **identical across all stacks** — copy verbatim:

- QUICK TRIGGERS
- Knowledge Base (3-Layer Loading, KB-First Rule)
- Mandatory Auto-Triggers (table + detection examples + correction phrases)
- Skills & Agents (full table)
- Self-Documentation Rules
- REMEMBER (End of File)

---

## Sections to customize per stack

### Project Context table

| Field | Guidance |
|-------|----------|
| Framework | Stack name + version hint (e.g. `Angular <!-- version -->`) |
| Language | Primary language (e.g. `TypeScript`, `Kotlin`, `Go`) |
| Styling | Common styling approaches for the stack |
| State Management | Common state solutions (e.g. `NgRx, Signals, RxJS services`) |
| Key commands | CLI commands: dev server, build, test, lint, generate |

### Critical Constraints

Write 5-7 constraints that are **universal** to the stack. Draw from:
- Type safety rules (language-specific)
- Component/module architecture norms
- Performance defaults (e.g. change detection strategy, lazy loading)
- Accessibility requirements
- Testing expectations
- Reactive/async patterns

### Stack-Specific Patterns

Include:
- **Project Structure** — idiomatic directory layout (as a code block)
- **Naming Conventions** — file naming, class naming, prefix/suffix conventions
- **Import Order** — standard import grouping for the language/framework

### Things to Avoid

Write 6-8 anti-patterns specific to the stack. Focus on:
- Common beginner mistakes
- Performance pitfalls
- Patterns superseded by modern framework features
- Code organization mistakes

### Quick Reference table

List the 6-8 most common developer commands for the stack's CLI/tooling.

### Layer 1 KB path

Update the Knowledge Base section to point to the correct stack:
```
.ai-framework/knowledge-base/{slug}/_index.md
```

### Technology choices hint

Update the "Optional: technology choices per topic" section with stack-relevant
examples of competing approaches (e.g. NgRx vs Signals vs RxJS services).
