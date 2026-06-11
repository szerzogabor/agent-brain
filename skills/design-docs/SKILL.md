---
name: design-docs
description: Write design documents and Architecture Decision Records (ADRs) - context, options considered, decision, consequences. Use when the user asks for a design doc, ADR, RFC, architecture proposal, or to "document this decision".
---

# Design Docs and ADRs

Capture a decision so well that a newcomer two years later understands what was decided, what the alternatives were, and why this one won.

## When to use

- A decision is expensive to reverse: architecture, data model, public API, dependency choice, build/deploy strategy.
- The user asks for a design doc, RFC, ADR, or proposal.
- Not for decisions that are cheap to change — those just need a code comment or commit message.

## Process

1. Identify the actual decision being made. One document per decision; if you are deciding three things, that is three ADRs or one design doc with three clearly separated decisions.
2. Gather context honestly: the problem, the constraints (technical, organizational, deadline), and the forces in tension. Write this section so that someone who disagrees with the decision still agrees with the problem statement.
3. List 2–4 real options, including the one you will reject. For each: a one-paragraph description, genuine pros, genuine cons. Steelman the rejected options — a design doc with one serious option and two strawmen convinces no one.
4. State the decision in one sentence, then the reasoning: which constraints dominated and why the trade-off lands this way.
5. Record the consequences, including the negative ones: what becomes harder, what is being deliberately given up, new risks accepted, and what would trigger revisiting the decision.
6. Keep it short. An ADR is typically one page; a design doc rarely needs more than three. Length is a cost paid by every future reader.

## Output format

For an ADR (single decision):

```markdown
# ADR-NNN: <decision title>
Date: YYYY-MM-DD | Status: proposed | accepted | superseded by ADR-MMM

## Context        <- the problem and constraints, neutrally stated
## Options        <- 2-4 options, each with pros/cons
## Decision       <- what was chosen, in one sentence, then why
## Consequences   <- positive AND negative outcomes; revisit triggers
```

For a larger design doc, add: **Goals / Non-goals** after Context, and a **Rollout / Migration** section before Consequences.

## Anti-patterns to avoid

- Writing the doc after the implementation to justify what was already built, while presenting it as open.
- Listing alternatives as strawmen so the favored option wins by default.
- Omitting the consequences section, or listing only the upsides.
- Burying the decision — a reader should find "what was decided" within ten seconds.
- Letting the doc rot: when a decision is reversed, mark the ADR superseded rather than editing history.
