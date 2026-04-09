# Compact Variant & Index Update Rules

Used by [add-kb-doc](../SKILL.md) Steps 5 and 6.

---

## Compact variant rules

If the full document is **100 lines or more**, create a compact variant.

- **Filename**: same as full doc but with `.compact.md` suffix (e.g. `combine-patterns.compact.md`)
- **Header**: `# [Topic] — Compact Reference` with a note to load the full doc for detailed examples
- **Content**: rules, constraints, and decision tables ONLY — no code examples
- **Target**: ~30% of the full doc's line count
- **Structure**: use bullet lists and tables instead of code blocks

Save to `knowledge-base/{STACK}/{category}/{filename}.compact.md`.

If the document is **under 100 lines**, skip — the full doc is already compact enough.

---

## Index update rules

Read `knowledge-base/{STACK}/_index.md` and add an entry for the new doc.

**Determine the loading tier:**

| Condition | Tier | Where in _index.md |
|-----------|------|-------------------|
| Core standards every task needs | Always Load | `## Always Load` section |
| Topic-specific, has compact variant | On-Demand with compact | Appropriate `## On-Demand` section, list both compact and full |
| Topic-specific, no compact variant (< 100 lines) | On-Demand full only | Appropriate `## On-Demand` section, list full doc only |

**Entry format (with compact):**

```markdown
| Topic (compact) | `{category}/{filename}.compact.md` | ~{lines} | {trigger keywords} |
| Topic (full) | `{category}/{filename}.md` | ~{lines} | Need detailed examples beyond compact |
```

**Entry format (without compact):**

```markdown
| Topic | `{category}/{filename}.md` | ~{lines} | {trigger keywords} |
```
