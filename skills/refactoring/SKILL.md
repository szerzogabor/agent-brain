---
name: refactoring
description: Behavior-preserving code restructuring in small verified steps. Use when the user asks to refactor, clean up, simplify, extract, rename, or restructure code without changing what it does.
---

# Refactoring

Change the structure of code without changing its behavior, in steps small enough that each one is obviously correct and independently verifiable.

## When to use

- The user asks to clean up, simplify, extract, inline, rename, or move code.
- Before adding a feature to code that resists the change ("make the change easy, then make the easy change").
- Not when behavior should change — that is a feature or fix, and mixing it with refactoring hides bugs.

## Process

1. Establish a safety net first: identify the tests covering the code to be refactored and run them to confirm they pass. If coverage is missing for the affected behavior, add characterization tests before restructuring anything.
2. Plan the end state, but execute as a sequence of small, named refactorings: extract function, inline variable, rename, move, replace conditional with polymorphism. Each step alone should leave the code working.
3. After each step, run the tests. If a step breaks them, revert that step rather than debugging forward — small steps make reverting cheap.
4. While restructuring, preserve observable behavior exactly: same outputs, same errors, same side effects, same performance characteristics where they matter. Resist "while I'm here" behavior tweaks; note them for a separate change.
5. Update all callers and references when renaming or moving — search the whole codebase, including tests, configuration, and documentation, not just compiler-checked code.
6. Prefer deleting code over generalizing it. Removed code is the cheapest code to maintain; speculative abstraction for hypothetical future needs is the most expensive.
7. Finish by re-reading the full diff: confirm it contains only structural change, and that the result is actually simpler (fewer concepts, clearer names, less duplication) — not just different.

## Output format

A diff containing only behavior-preserving changes, plus a short summary listing the refactorings applied and the verification run (test suite results).

## Anti-patterns to avoid

- Mixing behavior changes into a refactoring commit — reviewers cannot then verify either part.
- Big-bang rewrites when incremental steps were possible.
- Adding abstraction layers, interfaces, or configuration for needs that do not exist yet.
- Renaming things to a personally preferred vocabulary that conflicts with the project's existing naming.
- Refactoring without running tests because the change "is obviously safe".
