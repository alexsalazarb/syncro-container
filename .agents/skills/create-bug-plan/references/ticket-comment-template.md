# Ticket Comment Template

If a ticket was provided, compose a consolidated comment with:

- Root cause summary
- **How it was introduced** (from Step 2a) — system-focused, neutral framing:
  - "The fix for {ticket} correctly addressed {problem}; as a consequence, {side effect} was introduced."
  - "This scenario wasn't covered when {feature} was implemented."
- Feature history context and whether the original requirement is still valid
- Affected systems table
- Test gap and what the regression test will cover
- Fix plan table (task, project, status)
- **Domain expert** — who has context and can be consulted
- Any blockers or open decisions

**Tone**: write for the next person who reads this ticket in three months. Reference commits and code, not individuals. Assume every prior change was the best decision with the available information.

If Jira MCP is available: post directly via `addCommentToJiraIssue`.
If not: include the full comment text under `## Ticket Comment (ready to post)` in your report.
