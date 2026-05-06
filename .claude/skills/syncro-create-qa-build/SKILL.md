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

- **MUST be on `develop`** — abort if current branch is anything else
- **Only increment the build number** (the part after `+`) — never touch major.minor.patch
- **Commit message is fixed**: `Set version to x.x.x+xx` — no deviations
- **Force-push to BOTH remotes**: `origin` AND `bla` — never skip either one
- **Push BOTH branches**: `develop` AND `qa` to both remotes
- **qa must mirror develop exactly** — always rebase, never merge

## Steps

### 1. Verify Branch

```bash
git branch --show-current
```

If not `develop` → abort and tell the user:
> "Tenés que estar en el branch `develop` para crear un nuevo build."

### 2. Read and Increment Build Number

File: `syncro-flutter/pubspec.yaml`

Find the line matching `version: x.x.x+N`. Split on `+`:
- Left side (semantic version) → unchanged
- Right side (build number) → increment by 1

Write the updated line back.

**Example:** `version: 1.5.0+415` → `version: 1.5.0+416`

### 3. Commit on develop

```bash
git add syncro-flutter/pubspec.yaml
git commit -m "Set version to {new_version}"
```

Where `{new_version}` is the full version string, e.g. `1.5.0+416`.

### 4. Switch to qa and Rebase

```bash
git checkout qa
git rebase develop
```

### 5. Force Push to Both Remotes

Push `qa` first, then return to `develop` and push it too:

```bash
git push --force origin qa
git push --force bla qa
git checkout develop
git push --force origin develop
git push --force bla develop
```

## Commands

```bash
git branch --show-current
git add syncro-flutter/pubspec.yaml
git commit -m "Set version to {new_version}"
git checkout qa
git rebase develop
git push --force origin qa
git push --force bla qa
git checkout develop
git push --force origin develop
git push --force bla develop
```

## Output

Report to user:
- New version string (e.g. `1.5.0+416`)
- Confirmation that force push completed on both `origin` and `bla`
