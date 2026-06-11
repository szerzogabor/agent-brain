---
name: skill-name-in-kebab-case
description: One or two sentences describing what this skill does and, critically, WHEN an agent should use it. Include trigger phrases a user might say, e.g. "Use when the user asks to review code, check a PR, or says 'look over my changes'."
---

# Skill Title

One short paragraph: what this skill accomplishes and the outcome it produces.

## When to use

- Concrete situation 1 (e.g. "the user asks for a code review")
- Concrete situation 2
- When NOT to use it, if there is a common confusion with another skill

## Process

1. First step, written as an imperative instruction.
2. Second step. Keep steps capability-level — "search the codebase for X", not tool-specific commands — so any agent (Claude Code, Codex, Copilot, Gemini) can follow them.
3. Include decision points: "If X, do A; otherwise do B."
4. End with a verification step: how the agent confirms the work is correct.

## Output format

Describe exactly what the final deliverable looks like: structure, sections, length, tone. Provide a short example if the format is non-obvious.

## Anti-patterns to avoid

- Common mistake 1 and why it is wrong.
- Common mistake 2.
