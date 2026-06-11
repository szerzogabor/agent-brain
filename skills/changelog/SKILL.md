---
name: changelog
description: Maintain a Keep a Changelog-style CHANGELOG with user-facing, categorized entries. Use when the user asks to update the changelog, write release notes, or summarize what changed in a release or version.
---

# Changelog

Tell users what changed in language about THEIR experience, not about the code. A changelog entry answers: "what is different for me if I upgrade?"

## When to use

- Updating a CHANGELOG file for a release or as changes land.
- Writing release notes from a set of commits or merged PRs.

## Process

1. Follow the Keep a Changelog conventions: one `CHANGELOG.md`, newest version first, an `[Unreleased]` section at the top collecting changes between releases, each release as `## [X.Y.Z] - YYYY-MM-DD`.
2. Gather the actual changes since the last release from version history and merged PRs — do not write entries from memory.
3. Filter to what users can observe. Internal refactors, CI tweaks, and test-only changes do not belong unless they change behavior, performance, or requirements.
4. Categorize each entry under exactly one heading, in this order, omitting empty ones:
   - **Added** — new features.
   - **Changed** — changes to existing behavior.
   - **Deprecated** — still works, will be removed; say what to use instead.
   - **Removed** — gone in this release.
   - **Fixed** — bug fixes.
   - **Security** — vulnerability fixes (call these out even when embarrassing).
5. Write each entry as one user-facing line: what changed and, when not obvious, why it matters. Translate from implementation language — "Fixed crash when opening files with unicode names", not "Fixed NPE in FileNameParser.normalize()". Link the issue or PR number where the project does so.
6. Mark breaking changes loudly: a **BREAKING** prefix on the entry plus migration instructions (what the user must change, before/after example if short).
7. If the version number is part of the task, derive it from the content: breaking → major, added → minor, fixed only → patch (semantic versioning).

## Output format

```markdown
## [Unreleased]

## [1.4.0] - 2026-06-11
### Added
- Export reports as CSV from the dashboard (#142)
### Changed
- **BREAKING**: `--config` now expects a path to a file, not a directory. Pass `--config ./app.toml` instead of `--config ./`.
### Fixed
- Crash when opening files with unicode names (#155)
```

## Anti-patterns to avoid

- Dumping raw commit messages as the changelog.
- Entries only the authoring team understands ("refactored adapter layer").
- Hiding breaking changes under "Changed" without flagging them.
- Vague entries: "various fixes and improvements" tells the user nothing.
