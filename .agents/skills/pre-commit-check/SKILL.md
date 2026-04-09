---
name: pre-commit-check
description: >-
  Validates staged files against AGENTS.md standards before any git commit.
  Catches debug statements, hardcoded config/URLs, and runs stack-specific
  linters or checks defined in AGENTS.md. Runs BEFORE git operations.
---

# Pre-Commit Check

## Context Required
LOW-CONTEXT: AGENTS.md only

## Triggers

### Automatic (MUST run BEFORE the git operation, not after)

Trigger when user says ANY of:
- "commit", "git commit", "create commit"
- "stage", "git add", "add these files", "add to staging"
- "prepare for commit", "ready to commit"
- "create a PR", "open a pull request"
- Any variation of committing or staging code

**IMPORTANT**: Run this check BEFORE executing git commit.

### Manual
- `/pre-commit-check`
- "run pre-commit-check"
- "validate my changes"
- "check before commit"

## Instructions

### Step 0: Load project context

Read `.ai-framework.config` and resolve per [`common/resolve-project-context.md`](../common/resolve-project-context.md):
- `STACK` — determines which language-specific checks to apply
- `PROJECT_DIR` / `PROJECT_CODE_ROOT` — if container layout, infer the active project from staged file paths
- `AGENTS_MD` — `{PROJECT_CONTEXT_ROOT}AGENTS.md`

Read `{AGENTS_MD}` for:
- Any project-specific pre-commit rules
- Known files to skip (config-only files, generated code, etc.)
- Custom linter commands

### Step 1: Get files to check

```bash
# If files are staged, check those:
git diff --cached --name-only

# If nothing staged yet, check tracked changes plus untracked:
{ git diff --name-only --diff-filter=d; git ls-files --others --exclude-standard; } | sort -u
```

### Step 2: Universal checks (all stacks)

For every file in the changeset:

**a. No hardcoded API URLs or environment strings**
- Flag `http://`, `https://` string literals in non-config source files
- Flag: `[HARDCODED_URL] file:line — use a configuration constant`

**b. No hardcoded secrets or credentials**
- Flag patterns like `apiKey = "..."`, `password = "..."`, `secret = "..."`, `token = "..."`
- Flag: `[HARDCODED_SECRET] file:line — use environment variables or a secrets manager`

### Step 3: Stack-specific checks

#### Swift / iOS
Apply all checks from `knowledge-base/swift/ai-patterns/pre-commit-swift-rules.md`:
- Force unwraps (`!`, `as!`, `try!`)
- UI thread violations
- Hardcoded config values
- Debug print statements (`print(`, `NSLog(`)
- RxSwift: DisposeBag, `[weak self]`, `[unowned]`
- New `.swift` files not added to `.xcodeproj`
- `.pbxproj` / `Podfile.lock` warnings
- Localization checker if `.strings` files staged
- SwiftLint if available

#### Ruby / Rails
- Flag `puts`, `p` in non-test files
- Flag `binding.pry`, `byebug`, `debugger` in production code
- Flag: `[DEBUG_STMT] file:line — remove before shipping`
- Run `rubocop --autocorrect-all` if available; report errors

#### JavaScript / TypeScript
- Flag `console.log(`, `console.debug(`, `console.info(` in non-test files
- Flag `debugger;` statements
- Flag: `[DEBUG_STMT] file:line — remove before shipping`
- Run `eslint` on changed files if available; report errors

#### Python
- Flag `print(` in non-test files (unless `# noqa` suppressed)
- Flag `import pdb`, `pdb.set_trace()`, `breakpoint()`
- Flag: `[DEBUG_STMT] file:line — remove before shipping`
- Run `ruff` or `flake8` on changed files if available; report errors

#### Generic / Unknown stack
- Flag obvious debug output patterns (`console.log`, `print(`, `puts`, `System.out.println`)

### Step 4: `{AGENTS_MD}` custom rules

Check if `{AGENTS_MD}` defines a `Pre-commit checks` or `Before committing` section.
If so, apply those rules in addition to the above.

### Step 5: Report findings

```text
Pre-Commit Check Results
========================

Checking N files (stack: {STACK})...

VIOLATIONS FOUND:

[HARDCODED_URL] src/api/client.ts:14
  Literal URL in source — use API_BASE_URL from config

[DEBUG_STMT] src/components/Feed.tsx:87
  console.log found — remove before shipping

========================
N violations found.

Options:
1. Fix violations before committing
2. Proceed anyway (user must explicitly confirm)
```

### Step 6: Wait for user decision
- If violations: Ask "Fix these issues or commit anyway?"
- If clean: Proceed with commit

## Output

- Console report of all violations with file:line references
- Clear count of issues
- User prompted to fix or proceed
