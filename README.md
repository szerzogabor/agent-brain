# agent-brain

A portable library of AI coding-agent skills and personas. Write the methodology once, use it with **Claude Code, OpenAI Codex, GitHub Copilot, and Gemini CLI**.

Everything here is plain markdown with a small YAML frontmatter header — the canonical format is the Anthropic `SKILL.md` convention, which Claude Code consumes natively and every other agent can consume by inlining into its instruction file (`AGENTS.md`, `.github/copilot-instructions.md`, `GEMINI.md`).

## Layout

```
skills/      one folder per skill, each with a SKILL.md (methodology + checklists)
agents/      one file per persona (reusable system prompts / subagent definitions)
adapters/    wiring guide: how to plug this content into each agent
templates/   starting points for new skills and personas
```

## Skills

| Skill | What it gives the agent |
|---|---|
| [code-review](skills/code-review/SKILL.md) | Severity-ranked review methodology: correctness first, nits last |
| [commit-and-pr](skills/commit-and-pr/SKILL.md) | Atomic commits, Conventional Commits, structured PR descriptions |
| [debugging](skills/debugging/SKILL.md) | Reproduce → isolate → root-cause → fix → verify discipline |
| [test-writing](skills/test-writing/SKILL.md) | Behavior-named tests, AAA structure, edge cases, minimal mocking |
| [refactoring](skills/refactoring/SKILL.md) | Behavior-preserving steps, verified after each one |
| [task-planning](skills/task-planning/SKILL.md) | Risk-first breakdown into small, verifiable steps |
| [design-docs](skills/design-docs/SKILL.md) | ADR/design-doc structure: context, options, decision, consequences |
| [docs-writing](skills/docs-writing/SKILL.md) | Audience-first docs with verified, copy-pasteable examples |
| [changelog](skills/changelog/SKILL.md) | Keep a Changelog format, user-facing language, breaking-change flags |

## Personas

| Persona | Role |
|---|---|
| [code-reviewer](agents/code-reviewer.md) | Strict pre-merge reviewer; reports, never edits |
| [security-auditor](agents/security-auditor.md) | Defensive, exploitability-ranked security review |
| [architect](agents/architect.md) | Read-only planner; trade-offs and step-by-step plans |
| [debugger](agents/debugger.md) | Evidence-based root-cause hunter |
| [docs-writer](agents/docs-writer.md) | Audience-first technical writer |

## Quick start

Full instructions per agent are in [adapters/README.md](adapters/README.md). The short version:

- **Claude Code** — symlink/copy `skills/*` into `~/.claude/skills/` and `agents/*.md` into `~/.claude/agents/`. Done; skills auto-trigger.
- **Codex** — inline the skill bodies you need into your repo's `AGENTS.md`.
- **GitHub Copilot** — inline into `.github/copilot-instructions.md` (repo-wide) or `.github/instructions/*.instructions.md` (path-scoped); personas as chat modes.
- **Gemini CLI** — inline into `GEMINI.md`, or set `"contextFileName": "AGENTS.md"` and maintain one file for everything.

## Adding a skill or persona

1. Copy the matching file from [templates/](templates/) into `skills/<name>/SKILL.md` or `agents/<name>.md`.
2. Keep the body agent-agnostic — capability-level instructions only, no tool names from any specific agent.
3. Make the frontmatter `description` trigger-rich: what it does **and** when to use it.
4. Add a row to the table above.

Conventions are spelled out in [AGENTS.md](AGENTS.md) — which is also a live example of wiring this repo to agents.
