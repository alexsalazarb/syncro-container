---
name: check-test-coverage
description: >-
  Maps source files to expected test files based on conventions in AGENTS.md.
  Identifies public methods and classes that lack test coverage. Reports gaps
  as a prioritized list for the test-writer agent. Use after implementing a
  feature or before creating a PR.
---

# Check Test Coverage

## Context Required
MEDIUM-CONTEXT: AGENTS.md, source files being checked

## Triggers

### Automatic
- None (manual only)

### Manual
- `/check-test-coverage`
- "check test coverage"
- "what's missing tests?"
- "find untested code"
- "run check-test-coverage"

## Instructions

### Step 0: Resolve project context

Read `.ai-framework.config` and resolve `STACK`, `PROJECT_CODE_ROOT`, and `PROJECT_DIR` per [`common/resolve-project-context.md`](../common/resolve-project-context.md).
Read `{AGENTS_MD}` for:
- `Source directory` — where production source files live
- `Test directory` — where test files live
- `Test file naming convention` — e.g., `{Name}Tests.swift`, `test_{name}.py`, `{name}.test.ts`
- `Files / paths to exclude from coverage` — generated code, config-only files, singletons

If AGENTS.md does not define these, fall back to stack defaults below.

### Step 1: Identify source files to check

If invoked after a feature/fix, use the recently changed files.
Otherwise, scan all source files in `{PROJECT_CODE_ROOT}` excluding paths marked as "no tests needed" in `{AGENTS_MD}`.

**Stack defaults (if `{AGENTS_MD}` is silent):**

| Stack | Source glob | Exclude |
|-------|-------------|---------|
| Swift | `**/*.swift` | `**/Config/`, `**/Generated/`, `AppDelegate.swift` |
| Rails | `app/**/*.rb` | `app/views/`, `config/`, `db/` |
| React/TS | `src/**/*.{ts,tsx}` | `src/**/*.d.ts`, `src/index.ts` |
| Python | `**/*.py` | `migrations/`, `settings*.py`, `manage.py` |

### Step 2: Map source files to test files

For each source file `{Name}.ext`, derive the expected test file path using `{AGENTS_MD}` convention.

**Stack defaults:**

| Stack | Test file pattern | Test directory |
|-------|-------------------|----------------|
| Swift | `{Name}Tests.swift` | `{TestTarget}/` mirroring source structure |
| Rails | `{name}_spec.rb` or `test_{name}.rb` | `spec/` or `test/` |
| React/TS | `{name}.test.{ts,tsx}` or `{name}.spec.{ts,tsx}` | Co-located or `__tests__/` |
| Python | `test_{name}.py` | `tests/` mirroring source structure |

```bash
# Generic coverage gap finder — adapt SOURCE_DIR, TEST_DIR, EXT, and TEST_SUFFIX
SOURCE_DIR="{PROJECT_CODE_ROOT}/src"
TEST_DIR="{PROJECT_CODE_ROOT}/tests"
EXT="ts"          # file extension
TEST_SUFFIX=".test.ts"

find "$SOURCE_DIR" -name "*.${EXT}" | while read FILE; do
    BASE=$(basename "$FILE" ".${EXT}")
    TESTS=$(find "$TEST_DIR" -name "${BASE}${TEST_SUFFIX}" 2>/dev/null)
    if [ -z "$TESTS" ]; then
        echo "MISSING: $FILE"
    else
        echo "COVERED: $FILE → $TESTS"
    fi
done
```

### Step 3: For covered files — check method coverage

For source files that have a test file, compare method counts as a rough heuristic:

```bash
# Count public/exported functions/methods in source:
# Swift:
grep -c "^\s*\(public\|internal\)\?\s*func " "$SOURCE_FILE"
# JS/TS:
grep -c "^\s*\(export\s\+\)\?\(async\s\+\)\?function\|^\s*\(export\s\+\)\?const.*=.*=>" "$SOURCE_FILE"
# Python:
grep -c "^\s*def " "$SOURCE_FILE"
# Ruby:
grep -c "^\s*def " "$SOURCE_FILE"

# Count test methods in test file:
# Swift: grep -c "func test" "$TEST_FILE"
# JS/TS: grep -c "\bit(\|test(\|it\.each(" "$TEST_FILE"
# Python: grep -c "def test_" "$TEST_FILE"
# Ruby: grep -c "\bit\b\|\bexample\b\|\bspecify\b" "$TEST_FILE"
```

Flag if ratio is low (< 1 test per 2 public methods as a rough heuristic).

### Step 4: Priority triage

Prioritize coverage gaps using `{AGENTS_MD}`'s `Critical files` section if present.

**Default priorities:**

| Priority | Type | Why |
|----------|------|-----|
| P1 | Business logic services / managers | High-risk, easy to regress |
| P1 | Persistence / storage layer | Data integrity critical |
| P2 | Data models with non-trivial logic | Parsing, validation |
| P2 | Files recently modified this session | Immediate regression risk |
| P3 | Networking / utility helpers | Usually tested indirectly |

For Swift-specific priority patterns, see `knowledge-base/swift/architecture/testing-patterns.md`.

### Step 5: Report

```text
Test Coverage Report
====================
Stack: {STACK}
Source files checked: N
Files with test coverage: M (X%)
Files missing tests entirely: K

MISSING TEST FILES (P1 — business logic):
- UserService.ts → UserService.test.ts

MISSING TEST FILES (P2 — models):
- Article.ts → Article.test.ts

LOW COVERAGE (test file exists but sparse):
- FeedViewModel.ts — 8 exported functions, 2 test cases

Suggested next step: Run /execute-task on test-writing tasks, or
ask the test-writer agent to create the highest-priority test files.
```

### Step 6: Offer next step

After reporting, offer:
```text
Would you like me to:
1. Create a plan to systematically add test coverage?
2. Ask the test-writer agent to create the highest-priority test files now?
3. Just show the report (done)?
```

## Output

- Coverage report with file-by-file status
- Priority-ordered list of gaps
- Suggested next action (test-writer or create-plan)
