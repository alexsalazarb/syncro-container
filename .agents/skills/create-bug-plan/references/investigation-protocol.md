# Investigation Protocol

This is the key differentiator from `/create-plan`. Before creating any tasks, run a thorough investigation:

1. **Check KB (3-layer)** — Match bug-area keywords against KB indexes:
   a. **Layer 1 — Framework**: Match against `{FRAMEWORK_KB_DIR}/_index.md` triggers. Load the compact doc if directly relevant.
   b. **Layer 2 — Container**: Check `{CONTAINER_KB_DIR}/README.md` for domain docs, API contracts, or data model specs the bug may violate.
   c. **Layer 3 — Project**: Check `{KB_DIR}/README.md` for architecture docs, integration notes, or known-issues.
2. **Explore the code** — Use the ticket hypothesis as a starting point. Follow the code path.
3. **Document findings** — For each suspected issue: file path, line numbers, what the code does vs. what it should do, why it's broken
4. **Validate or disprove the hypothesis** — State clearly whether the original hypothesis was confirmed, partially correct, or wrong
5. **Identify all affected systems** — List every component that touches the buggy code path

---

## Step 2a: Origin Analysis (Mandatory)

Once the buggy code is identified, trace how and why it got there using `git blame` and `git log`.

**Tone**: Origin analysis is about understanding the system's history. Reference commits and code — not individuals. Frame findings as system consequences, not personal mistakes.

1. Find the commit via `git blame` on the affected lines
2. Read the full commit message and diff
3. If linked to a prior ticket/PR, fetch and read it
4. Determine: what was the change solving, and how did this bug result as a consequence?
5. Document: commit hash, date, related ticket, chain of causation

**Origin classification** (use one):
- **Side effect of a prior change** — A previous fix introduced this as a consequence
- **Coverage gap** — A scenario not anticipated when the feature was built
- **Evolved from a refactor** — Code behavior changed during a refactor or merge
- **Pre-existing gap** — This area predates the feature or structured tracking
- **Configuration/data issue** — Code is correct; configuration or data is unexpected

---

## Step 2b: Feature History Trace (Mandatory, skip for P0)

Trace the full history of the feature area the bug affects. This prevents fixing symptoms while missing context that changes the fix approach.

1. **Find the original feature commit/PR** from the origin trace
2. **Scan `.plans/` for related plans** — plans contain scope decisions and investigation artifacts:
   ```bash
   git log --all -- '.plans/'
   ```
3. **Build a brief timeline** — For each related ticket or plan: date, what changed, relevance to the current bug
4. **Document key takeaways** — What does the timeline reveal that affects our fix approach?

---

## Step 2c: Test Coverage Analysis (Mandatory)

Explain why existing tests didn't catch this bug:

1. **Find related tests** — Search for tests covering the buggy code path
2. **If tests exist**: determine whether they cover the triggering scenario; if yes, why did they pass?
3. **If no tests exist**: document the missing coverage area
4. **Document the gap** and write a specific recommendation for what the regression test should assert

---

## Step 2d: Cross-Project Assessment (Automatic)

Based on investigation findings, determine if the fix requires changes in multiple projects. This is discovered from the evidence — not a user flag.

**Indicators that a bug is cross-project:**
- Root cause is in one project but consumers in other projects need to handle the fix
- The fix requires coordinated changes (e.g., API response format change + client update)
- Investigation found affected systems in different project directories

**If cross-project is detected (container layout):**
1. Note which project directories are involved and what changes each needs
2. Flag this to the user: "This bug spans {projects} — I'll create a master plan to coordinate."
3. The master plan is created in Step 5b

**If single-project:** proceed normally, set `Master Plan` field to "None" in overview.

---

## Step 2e: Confidence Assessment (Mandatory)

| Confidence | Meaning | Action |
|------------|---------|--------|
| **High** (>85%) | Root cause identified, code path confirmed, evidence aligns | Proceed to task design |
| **Medium** (50-85%) | Likely root cause, some uncertainty remains | Present findings with uncertainty flags. Ask user to confirm before proceeding. |
| **Low** (<50%) | Hypothesis disproved or multiple competing root causes | **Stop.** Present what you found, what you ruled out, what information is needed. |

Set the `Root Cause Confidence` field in `investigation.md`. If Medium or Low, document what would raise confidence.
