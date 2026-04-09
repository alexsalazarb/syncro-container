# KB Document Template & Topic Standards

Used by [add-kb-doc](../SKILL.md) Steps 2b and 4.

---

## Topic-standard awareness

After choosing the category, check whether this doc covers a **technology-choice topic** —
one where multiple alternative approaches exist for the same problem (e.g. concurrency
strategies, persistence layers, state management, navigation patterns, networking approaches).

**Signs a topic is a technology-choice topic:**
- The doc title contains words like "approach", "pattern", "strategy", "using X"
- The category is `architecture/` or `integrations/`
- An alternative approach exists (or could exist) in the same category

**If the topic is a technology-choice topic**, note the following before writing:

> ⚠️ **Technology-choice topic detected.**
> This doc covers one of potentially several approaches for a given problem area.
> When updating `_index.md` (Step 6), organize the entry to make the alternatives
> discoverable. Recommended `_index.md` format:
>
> ```markdown
> <!-- Concurrency — multiple approaches available -->
> | Async/Await (compact) | `architecture/async-await.compact.md` | ~25 | async, await, concurrency, structured concurrency |
> | Async/Await (full)    | `architecture/async-await.md`         | ~90 | Need detailed examples beyond compact |
> | Combine (compact)     | `architecture/combine-patterns.compact.md` | ~20 | combine, publisher, subscriber, reactive |
> | Combine (full)        | `architecture/combine-patterns.md`    | ~75 | Need detailed examples beyond compact |
> ```
>
> If a `docs/KB_TOPIC_STANDARDS.md` file exists in the framework, read it for the
> full conventions on organizing competing-approaches docs in `_index.md`.

If `docs/KB_TOPIC_STANDARDS.md` does not yet exist in the framework, proceed without it —
the inline guidance above is sufficient.

---

## Document template

Use this structure for all new KB documents:

```markdown
# [Topic Title]

**Last Updated**: [Month Year]
**Context**: Read when [trigger keywords — be specific so _index.md can route accurately].

---

## Overview

[1-2 paragraphs: what this covers and when it matters]

---

## [Core Section(s)]

[Rules, patterns, and code examples organized by subtopic]

### [Pattern/Rule Name]

[Explanation + correct/incorrect code examples in the stack's language]

---

## Gotchas

[Non-obvious pitfalls and edge cases]

---

## Related

- [Links to related KB docs in the same stack]
```

Save to `knowledge-base/{STACK}/{category}/{filename}.md`.
