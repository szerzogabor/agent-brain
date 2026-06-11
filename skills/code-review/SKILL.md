---
name: code-review
description: Review code changes for correctness, security, and maintainability with severity-ranked, actionable findings. Use when the user asks to review code, check a PR or diff, says "look over my changes", or before merging a branch.
---

# Code Review

Produce a review that catches real bugs first and style issues last, with every finding actionable enough that the author knows exactly what to change.

## When to use

- The user asks to review a PR, branch, diff, or specific files.
- Before merging or as a pre-commit sanity check.
- Not for deep security audits — use the security-auditor persona for that.

## Process

1. Establish context: read the PR description or ask what the change is supposed to do. A review without intent is just style policing.
2. Read the full diff once end-to-end before commenting on anything.
3. Hunt for correctness bugs in priority order:
   - Logic errors: inverted conditions, off-by-one, wrong operator, unhandled null/empty/error cases.
   - Concurrency and state: race conditions, shared mutable state, missing cleanup.
   - Contract violations: does the change break callers? Search the codebase for usages of every modified public function or interface.
   - Edge cases: empty inputs, large inputs, unicode, time zones, error paths.
4. Check security basics: injection (SQL, command, path), secrets in code, missing input validation at trust boundaries, unsafe deserialization.
5. Check tests: do they exist for the new behavior, and would they actually fail if the code were wrong?
6. Only after the above, note maintainability issues: duplication, naming, dead code, missing comments on non-obvious constraints.
7. Verify your own findings: before reporting a bug, re-read the surrounding code to confirm it is real, not a misreading. Drop anything you are not confident about or label it explicitly as a question.

## Output format

- Start with a one-paragraph verdict: ship it / fix issues first / needs rework, and why.
- List findings ordered by severity, each tagged:
  - `[blocker]` — bug or security issue that must be fixed before merge.
  - `[important]` — likely problem or significant maintainability cost.
  - `[nit]` — optional polish; the author may ignore these.
- Each finding: file and line reference, what is wrong, why it matters, and a concrete suggested fix.
- If the change is clean, say so plainly — do not invent findings to look thorough.

## Anti-patterns to avoid

- Reporting style nits with the same weight as bugs.
- Vague feedback like "consider improving error handling" without saying where and how.
- Reviewing only the diff lines while ignoring how the change interacts with unchanged code.
- Suggesting rewrites in the reviewer's preferred style when the existing approach is correct.
