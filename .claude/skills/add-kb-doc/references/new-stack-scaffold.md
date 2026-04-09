# Creating a New Stack KB

Used by [add-kb-doc](../SKILL.md) when the target stack has no KB directory yet.

---

## Directory structure

Create the following:

```
knowledge-base/{stack}/
├── _index.md
├── README.md
└── best-practices/
    └── {stack}-standards.md
```

## Seed `_index.md`

```markdown
# {Stack} KB — Quick Index

Load this file first. Load full docs **only** when the task matches the triggers below.

## Always Load (project conventions)

| Doc | Path | Lines | Triggers |
|-----|------|-------|----------|
| {Stack} Standards | `best-practices/{stack}-standards.md` | ~{N} | Any {stack} code change |

## Loading Rules

1. **Read this index** before loading any KB doc
2. **Match task keywords** against the Triggers column
3. **Prefer compact** variants — only load full when you need detailed examples
4. **Skip irrelevant docs** — only load what the current task needs
5. As the KB grows, add new entries here with trigger keywords
```

## Seed `README.md`

Follow the format in `knowledge-base/swift/README.md`.

After scaffolding, proceed with Step 4 to write the actual doc.
