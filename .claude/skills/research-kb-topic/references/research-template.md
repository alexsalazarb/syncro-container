# Research Template

Procedure and quality criteria for `research-kb-topic` Step 4 (web research)
and Step 5 (content generation).

---

## Required Research Areas

For each technology, gather information across these dimensions:

| Area | What to find | Priority |
|------|-------------|----------|
| **Overview** | What it is, what problem it solves, when to use vs. alternatives | Required |
| **Setup & Configuration** | How to add it to a project, basic configuration | Required |
| **Core Patterns** | The 3-5 most common usage patterns with code examples | Required |
| **Architecture** | How it fits into app architecture (MVVM, Clean, etc.) | Required |
| **Testing** | How to test code that uses this technology (mocks, fakes, test utilities) | Required |
| **Migration** | Common migration paths (version upgrades, from/to alternatives) | If applicable |
| **Performance** | Performance considerations, memory management, threading | If applicable |
| **Gotchas** | Non-obvious pitfalls, common mistakes, sharp edges | Required |

---

## Search Query Patterns

Use these query templates with WebSearch. Replace `{tech}` and `{stack}`:

1. `"{tech} best practices {year}"` — current recommendations
2. `"{tech} architecture patterns {stack}"` — how to structure code
3. `"{tech} common mistakes"` or `"{tech} gotchas"` — pitfalls
4. `"{tech} testing {stack}"` — test strategies
5. `"{tech} vs {alternative}"` — positioning relative to alternatives
6. `"{tech} migration guide"` — version upgrade patterns

Use the **current year** in queries to get recent results. Follow up with WebFetch
on the most relevant documentation pages.

**Prioritize official documentation** over blog posts. If the technology has an
official guide or API reference, fetch those pages first.

---

## Quality Criteria

The generated KB doc MUST meet these minimums before delegating to `add-kb-doc`:

| Criterion | Minimum |
|-----------|---------|
| **Code examples** | At least 3 code snippets in the stack's language |
| **Correct API surface** | Verify API names against official docs (web research required) |
| **Gotchas section** | At least 2 non-obvious pitfalls |
| **Trigger keywords** | 5+ specific keywords for `_index.md` routing |
| **No hallucinated APIs** | Every method/class name must come from official docs or project code |
| **Actionable** | A developer reading this doc can start using the technology correctly |
| **Stack-generic** | No project-specific classes, business logic, or infrastructure references |

**If web research is insufficient** (technology is obscure, docs are sparse):
1. Note the limitation in the doc's Overview section
2. Mark the doc as `Draft` in the title: `# [Topic] (Draft)`
3. Still create the doc — partial coverage is better than none
4. Suggest the user review and enhance the doc with project-specific experience

---

## Content Generation Order

Generate sections in this order for best results:

1. **Trigger keywords first** — decide what tasks should load this doc
2. **Overview** — position the technology (what, when, why)
3. **Core patterns** — the 3-5 things every developer needs to know
4. **Code examples** — concrete, runnable snippets (verify against official docs)
5. **Gotchas** — what trips people up
6. **Related** — link to other KB docs in the same stack

This order front-loads the most important decisions (routing and scope) before
diving into content generation.
