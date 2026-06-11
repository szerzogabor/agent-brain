---
name: commit-and-pr
description: Write atomic commits with Conventional Commits messages and structured PR descriptions. Use when committing changes, preparing a pull request, or when the user asks to "commit this", "open a PR", or "write a commit message".
---

# Commits and Pull Requests

Produce commits a reviewer can follow change-by-change and PR descriptions that explain why, not just what.

## When to use

- Any time changes are being committed or a pull request is being prepared.
- When the user asks to split, squash, or clean up a messy set of changes.

## Process

1. Review what actually changed (diff of staged and unstaged work) before writing anything. Never describe changes from memory.
2. Group changes into atomic commits: each commit should compile and pass tests on its own, and do exactly one thing. If the diff mixes a refactor with a feature, split them.
3. Write the commit message in Conventional Commits format:
   - `type(scope): summary` — types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`, `build`.
   - Summary line: imperative mood ("add", not "added"), no trailing period, under 72 characters.
   - Body (when the change is non-trivial): explain WHY the change was made and any non-obvious decisions. The diff already shows what changed.
   - Footer: `BREAKING CHANGE:` description when applicable; issue references like `Closes #123`.
4. For the PR description, use this structure:
   - **Summary** — 1–3 sentences: the problem and the approach taken.
   - **Changes** — bullet list of the meaningful changes (not a file list).
   - **Why** — context a reviewer needs: alternatives considered, constraints, links to issues or design docs.
   - **Testing** — what was run to verify, and how a reviewer can verify locally.
   - **Risks / Rollback** — anything that could break, and how to undo it (omit if genuinely trivial).
5. Before finalizing, re-read the message against the diff: does it cover everything, and nothing that is not there?

## Anti-patterns to avoid

- Messages describing the act of coding ("fixed stuff", "address review comments") instead of the change.
- One giant commit mixing unrelated changes, or the opposite: ten commits that only make sense together.
- PR descriptions that paraphrase the diff line-by-line and never explain motivation.
- Claiming "tested" without saying what was actually run.
