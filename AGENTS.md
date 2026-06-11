# agent-brain — instructions for agents working in this repo

This repository is a library of agent-agnostic skills and personas. It contains no application code — the markdown files ARE the product. Treat content edits with the care you would give source code.

## Layout

- `skills/<name>/SKILL.md` — one skill per folder: YAML frontmatter (`name`, `description`) + imperative markdown body.
- `agents/<name>.md` — one persona per file, same frontmatter format; body is a system prompt.
- `templates/` — start every new skill or persona from the matching template.
- `adapters/README.md` — how to wire this content into Claude Code, Codex, Copilot, and Gemini.

## Conventions (enforced)

1. **Agent-agnostic bodies.** Never reference a specific agent's tool names in a skill or persona body ("use the Grep tool", "run the Bash tool"). Write capability-level instructions: "search the codebase for…", "run the tests". The adapters guide is the only place agent-specific details belong.
2. **Trigger-rich descriptions.** The frontmatter `description` must say what the skill does AND when to use it, including phrases a user might actually say — agents select skills by this field alone.
3. **Skill structure.** Follow `templates/SKILL-template.md`: When to use → Process → Output format (if relevant) → Anti-patterns. Keep bodies roughly 40–80 lines.
4. **Persona structure.** Follow `templates/agent-template.md`: focus → how you work → what you avoid → output. Roughly 20–40 lines.
5. **Kebab-case names** everywhere; folder name must equal the frontmatter `name`.
6. When adding or renaming a skill/persona, update the tables in `README.md`.

## Verification before committing

- Every `skills/*/SKILL.md` and `agents/*.md` has frontmatter with non-empty `name` and `description`.
- No agent-specific tool names in bodies (search for "Grep tool", "Bash tool", "Edit tool", and similar).
- All relative links in `README.md` and `adapters/README.md` resolve.
