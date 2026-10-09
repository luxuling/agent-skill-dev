# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Source for `code-standards`, a personal Agent Skill that makes AI coding agents write code the owner's way. There is no application code and no build step. The deliverable is the `skills/code-standards/` folder, published from GitHub as `luxuling/agent-skill-dev`.

The rules in the skill come from the owner, who was interviewed for them. Do not add, drop, or soften a rule on your own judgment. If a case is not covered, ask the owner and write down their answer.

## Layout

- `skills/code-standards/SKILL.md` holds the frontmatter (what triggers the skill) and the language-agnostic rules: comments, error handling, defensive code, abstraction, naming, folder layout, testing, blank lines, formatting, git.
- `skills/code-standards/references/` holds stack-specific rules that `SKILL.md` tells the agent to read only for that stack, so other work does not pay for them. `typescript.md` covers TypeScript, React, Next.js, and Node. `tailwind.md` covers Tailwind classes. Add another language or tool as a new file here plus a pointer line in `SKILL.md`.
- `skills/code-standards/evals/evals.json` holds the test prompts used to check the skill.
- `.claude-plugin/marketplace.json` lists the skill as a Claude Code plugin. Its `skills` path must point at the skill folder.

The skill lives under `skills/` because that is where the `skills` CLI (`npx skills add`) looks for it. Keep it there.

## Format constraints

The skill follows the open Agent Skills spec (https://agentskills.io/specification) so that one folder works in Claude Code, Codex, Gemini CLI, Cursor, and other agents. No per-agent wrapper files are needed.

- `name` in the frontmatter must equal the directory name: lowercase letters, digits, and single hyphens.
- `description` is at most 1024 characters. It is the only text an agent sees when deciding whether to load the skill, so anything about when to use the skill goes there and not in the body.
- Keep `SKILL.md` under 500 lines, and keep reference files one level below it.

## Install

Users install with `npx skills add luxuling/agent-skill-dev` or through the Claude Code plugin marketplace (see `README.md`). For local development, symlink the folder so edits here take effect without copying:

```sh
ln -s "$PWD/skills/code-standards" ~/.claude/skills/code-standards   # Claude Code
ln -s "$PWD/skills/code-standards" ~/.agents/skills/code-standards   # Codex, Gemini CLI, others
```

## Testing a change

Use the `skill-creator` skill. It runs each prompt in `skills/code-standards/evals/evals.json` twice, once with the skill and once without, and opens a viewer to compare the outputs. Results go in `skills/code-standards-workspace/iteration-N/`, a sibling of the skill folder that is not part of the skill. It is listed in `.gitignore` so it is never published.
