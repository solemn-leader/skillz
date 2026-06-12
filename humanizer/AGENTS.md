# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Warp, etc.) working in this repository.

## What this repo is

A **Claude Code / OpenCode skill** implemented entirely as Markdown. The runtime artifact is `SKILL.md`: the agent reads its YAML frontmatter (metadata + allowed tools) followed by the editor prompt. There is no build step and no code to run.

## Key files

- `SKILL.md` — the skill itself. YAML frontmatter (`name`, `version`, `description`, `allowed-tools`) followed by the canonical, numbered pattern list with before/after examples. **This is the source of truth.**
- `README.md` — for humans: installation, usage, a summary of the patterns.

## The maintenance contract

`SKILL.md` and `README.md` must stay in sync. When you change behavior or content:

- **Patterns:** the skill currently defines **33 numbered patterns**. If you add, remove, or renumber any, update the README pattern summary and every cross-reference in the same change. Keep numbering stable unless you are deliberately renumbering.
- **Version:** `SKILL.md` frontmatter has a `version:` field. Bump it when behavior changes.
- **Non-obvious fixes:** if you change the prompt to handle a tricky failure mode (a repeated mis-edit, an unexpected tone shift), note what was fixed and why.

## Editing SKILL.md

- Preserve valid YAML frontmatter (formatting and indentation).
- The prompt below the frontmatter is the product. Edit it like a careful instruction document, not code.
