---
name: docs-writing
description: Write user-facing documentation - READMEs, guides, API docs - audience-first with working examples. Use when the user asks to write or improve a README, documentation, usage guide, or onboarding material.
---

# Documentation Writing

Write documentation that gets a specific reader to a specific outcome with the least reading possible. Every section must answer a question a real reader actually has.

## When to use

- Creating or improving READMEs, usage guides, API references, onboarding docs.
- Not for design decisions (use the design-docs skill) or release notes (use the changelog skill).

## Process

1. Name the audience and their goal before writing a word: "a developer who wants to install and run this in 5 minutes" reads very differently from "a contributor who wants to modify the parser". If the doc serves both, structure it so each can skip the other's parts.
2. Read the actual code before documenting it. Verify every command, flag, path, and code sample against the current state of the project — documentation that lies is worse than none.
3. Lead with the most common path: what it is (one sentence), then install, then the smallest working example. Reference material, edge cases, and configuration tables come after.
4. Show, then tell: a working, copy-pasteable example first, prose explanation second. Examples must be complete enough to run as-is — no `<your-value-here>` placeholders where a real default would work.
5. For a README specifically, the canonical order is: one-line description → badges (optional) → quick start (install + minimal usage) → key features or concepts → configuration → contributing/development setup → license. Cut any section with nothing real to say.
6. Write plainly: short sentences, active voice, second person ("run the command", not "the command may be run"). Define a term at first use or do not use it.
7. Verify before finishing: run every command and code sample in the doc, and click every link. Anything you did not verify, do not ship.

## Anti-patterns to avoid

- Documenting the code's structure instead of the reader's task ("this module contains..." vs "to do X, call...").
- Examples that have drifted from the code and no longer run.
- Walls of prose where a list, table, or example would be scanned in seconds.
- Marketing language ("blazingly fast", "simple yet powerful") in place of facts.
- Writing for the author's knowledge level instead of the stated audience's.
