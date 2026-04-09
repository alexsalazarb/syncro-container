# INDEX.md Section Formats

Use the correct column format for each state section when updating `$HUB_ROOT/plans/INDEX.md`.

## Active Plans

```markdown
| {slug} | {title} | {type} | {status} | {owner} | {target_demo} | {repos} |
```

## Planned

```markdown
| {slug} | {title} | {type} | {priority} | {owner} | {target_demo} | {repos} |
```

## Backlog

```markdown
| {slug} | {title} | {type} | {owner} | {YYYY-MM-DD} |
```

## Completed

```markdown
| {slug} | {title} | {type} | {YYYY-MM-DD} | {owner} |
```

## Deprioritized

```markdown
| {slug} | {title} | {type} | {reason} | {YYYY-MM-DD} |
```

## Cancelled

```markdown
| {slug} | {title} | {type} | {reason} | {YYYY-MM-DD} |
```

Read field values from `plan.md` (title, type, owner, priority, target demo) and STATUS.md (repos list). Use today's date for date columns in completed/deprioritized/cancelled sections.
