---
name: task-planning
description: Break work into small, verifiable, ordered steps with risks surfaced first. Use when planning a feature or migration, when the user asks "how should we approach this", "make a plan", or before starting any task too large to hold in your head.
---

# Task Planning

Turn a goal into an ordered list of steps where each step is independently verifiable, and the riskiest unknowns are resolved first.

## When to use

- Before implementing anything that touches more than a couple of files or has unknowns.
- When the user asks for an approach, estimate, breakdown, or implementation plan.
- Not for trivial single-edit tasks — just do those.

## Process

1. Restate the goal in one sentence, including the definition of done. If you cannot, the task is not understood yet — ask or investigate before planning.
2. Investigate before writing the plan: read the relevant code, find existing utilities and patterns that can be reused, and identify what the change actually touches. A plan written without looking at the code is fiction.
3. List the unknowns and risks explicitly: unfamiliar subsystems, ambiguous requirements, external dependencies, migration hazards. For each, note how it will be resolved (a question to the user, a spike, a test).
4. Order the steps risk-first: tackle the step most likely to invalidate the plan early, when changing course is cheap. Leave mechanical, low-risk work for last.
5. Size each step so it is completable and verifiable on its own — roughly "one commit" granularity. For every step, state how you will know it worked (a test passes, the app runs, output matches).
6. Identify what is explicitly out of scope. A plan without a non-goals line tends to grow silently.
7. Sanity-check the plan: if every step succeeds, is the goal met? Is any step doing two unrelated things? Could any two steps be merged or one deleted?

## Output format

- **Goal:** one sentence with the definition of done.
- **Context:** what was learned from investigating the code (key files, reusable pieces).
- **Risks / open questions:** bullets, each with a resolution path.
- **Steps:** numbered, each with a one-line description and its verification.
- **Out of scope:** bullets.

Keep the whole plan scannable — if it does not fit on roughly one screen, the task probably needs to be split.

## Anti-patterns to avoid

- Planning by enthusiasm: starting with the easy, satisfying steps and deferring the risky unknown to step 9.
- Steps with no observable completion criterion ("improve the architecture").
- A plan so detailed it is really the implementation written twice.
- Skipping investigation and planning against an imagined codebase.
