---
name: syncro-create-qa-build
description: >
  Increments the build number in pubspec.yaml, commits on develop, rebases qa from develop,
  and force-pushes to all remotes (origin + bla).
  Trigger: /syncro-create-qa-build or "create new build", "nuevo build", "crear build".
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

- User explicitly says `/syncro-create-qa-build`, "create new build", "nuevo build", "crear build", or similar
- Do NOT trigger automatically — requires an explicit request

## Critical Patterns

- **ALL git operations run inside `syncro-flutter/`** — this is a subproject with its own git repo
- **MUST be on `develop`** — check `syncro-flutter` branch, NOT the container repo branch
- **Only increment the build number** (the part after `+`) — never touch major.minor.patch
- **Commit message is fixed**: `Set version to x.x.x+xx` — no deviations
- **Force-push to BOTH remotes**: `origin` AND `bla` — never skip either one
- **Push BOTH branches**: `develop` AND `qa` to both remotes
- **qa must mirror develop exactly** — always rebase, never merge

## Steps

### 1. Verify Branch (inside syncro-flutter)

```bash
# From container root — syncro-flutter is a separate git repo
git -C syncro-flutter branch --show-current
```

If not `develop` → switch to it:
```bash
git -C syncro-flutter checkout develop
```

### 2. Read and Increment Build Number

File: `syncro-flutter/pubspec.yaml` (read from container root)

Find the line matching `version: x.x.x+N`. Split on `+`:
- Left side (semantic version) → unchanged
- Right side (build number) → increment by 1

Write the updated line back.

**Example:** `version: 1.5.0+415` → `version: 1.5.0+416`

### 3. Commit on develop (inside syncro-flutter)

```bash
git -C syncro-flutter add pubspec.yaml
git -C syncro-flutter commit -m "Set version to {new_version}"
```

Where `{new_version}` is the full version string, e.g. `1.5.0+416`.

### 4. Switch to qa and Rebase (inside syncro-flutter)

```bash
git -C syncro-flutter checkout qa
git -C syncro-flutter rebase develop
```

### 5. Force Push to Both Remotes (inside syncro-flutter)

Push `qa` first, then return to `develop` and push it too:

```bash
git -C syncro-flutter push --force origin qa
git -C syncro-flutter push --force bla qa
git -C syncro-flutter checkout develop
git -C syncro-flutter push --force origin develop
git -C syncro-flutter push --force bla develop
```

## Commands

```bash
git -C syncro-flutter branch --show-current
git -C syncro-flutter add pubspec.yaml
git -C syncro-flutter commit -m "Set version to {new_version}"
git -C syncro-flutter checkout qa
git -C syncro-flutter rebase develop
git -C syncro-flutter push --force origin qa
git -C syncro-flutter push --force bla qa
git -C syncro-flutter checkout develop
git -C syncro-flutter push --force origin develop
git -C syncro-flutter push --force bla develop
```

## Output

Report to user:
- New version string (e.g. `1.5.0+416`)
- Confirmation that force push completed on both `origin` and `bla`
