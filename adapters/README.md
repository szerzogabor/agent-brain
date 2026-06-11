# Wiring guide: using agent-brain with each agent

Every skill in this repo is a folder under [`skills/`](../skills/) containing a `SKILL.md` (YAML frontmatter + plain markdown body). Every persona is a single file under [`agents/`](../agents/) in the same format. Both are deliberately plain markdown, so any agent that reads instruction files can consume them — the only per-agent work is telling each tool where to look.

> **Lowest-maintenance path:** most modern agents support a root `AGENTS.md` (Codex natively, GitHub Copilot since 2025, Gemini CLI via the `contextFileName` setting, Claude Code reads it too). Keeping a thin `AGENTS.md` in your target repo that links to or inlines these skills covers everything with one file. The per-agent options below give you tighter, native integration where it exists.

## Claude Code

Skills are natively supported — no inlining needed.

- **Personal (all projects):** copy or symlink skill folders into `~/.claude/skills/`:

  ```powershell
  # PowerShell (run as admin or with Developer Mode for symlinks)
  Get-ChildItem .\skills -Directory | ForEach-Object {
    New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills\$($_.Name)" -Target $_.FullName
  }
  ```

- **Per project:** copy skill folders into `<repo>/.claude/skills/`.
- **Personas as subagents:** copy `agents/*.md` into `<repo>/.claude/agents/` (project) or `~/.claude/agents/` (personal). Claude Code reads the same frontmatter format directly; optionally add a `tools:` key to restrict tool access (e.g. `tools: Read, Grep, Glob` for the read-only architect/reviewer).

## OpenAI Codex

Codex reads `AGENTS.md` at the repo root (and `~/.codex/AGENTS.md` globally).

- Inline the body of the skills you want into `AGENTS.md`, or keep `AGENTS.md` thin and paste the relevant `SKILL.md` body into the conversation when needed.
- Personas: use the body of an `agents/*.md` file as a custom instruction block or paste it at the start of a session ("Act as the following:").

## GitHub Copilot

- **Repo-wide instructions:** inline selected skill bodies into `.github/copilot-instructions.md`. Keep it focused — Copilot sends this with every request, so include only the always-relevant skills (e.g. commit-and-pr, code-review).
- **Path-scoped instructions:** create `.github/instructions/<name>.instructions.md` with an `applyTo:` frontmatter glob, e.g. apply `test-writing` only to test files:

  ```markdown
  ---
  applyTo: "**/*.test.ts"
  ---
  <body of skills/test-writing/SKILL.md>
  ```

- **Personas as chat modes:** copy an `agents/*.md` body into `.github/chatmodes/<name>.chatmode.md` (VS Code) with a `description:` frontmatter line.
- Copilot also reads root `AGENTS.md`.

## Gemini CLI

- Inline or reference skills in `GEMINI.md` at the repo root, or `~/.gemini/GEMINI.md` for global use. Gemini merges all `GEMINI.md` files found from the global dir down to the working directory.
- Alternatively, point Gemini at the shared file instead of maintaining a separate one — in `.gemini/settings.json`:

  ```json
  { "contextFileName": "AGENTS.md" }
  ```

- Personas: paste the persona body as the opening instruction of a session, or include it in `GEMINI.md` under a "When asked to review/debug/audit..." heading.

## General rules when wiring

- Inline the **body** of a `SKILL.md` (below the frontmatter) into instruction files; the frontmatter is metadata for tools like Claude Code that index skills.
- Do not inline everything everywhere. Instruction files are sent on every request — pick the 2–4 skills relevant to the repo and link the rest.
- When you update a skill here, re-sync any copies you made. Symlinks (Claude Code) avoid this; inlined copies (Copilot, Codex) do not — note the source path in a comment when you inline.
