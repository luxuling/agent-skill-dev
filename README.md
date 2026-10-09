# code-standards

![Agent Skills](https://img.shields.io/badge/Agent_Skills-spec-black)
![Claude Code](https://img.shields.io/badge/Claude_Code-ready-d97757)
![Codex](https://img.shields.io/badge/Codex-ready-10a37f)
![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-ready-4285f4)
![License: MIT](https://img.shields.io/badge/license-MIT-blue)

An [Agent Skill](https://agentskills.io/specification) that makes AI coding agents write code the owner's way: clean, readable, and consistent, with no manual cleanup afterwards.

One folder works in Claude Code, Codex, Gemini CLI, Cursor, and any other agent that supports the Agent Skills spec.

## What it enforces

| Area | Rule |
| --- | --- |
| Comments | None by default. Only `TODO`, `FIX`, `HACK`, `WARN`, `PERF`, `NOTE`, `TEST` markers |
| Errors | Throw, and catch only at the boundary |
| Defensive code | No needless null checks, fallback defaults, or optional chaining |
| Abstraction | One job per function, shared on the second use, components only render |
| Readability | Blank lines between logical steps, never one dense block |
| Naming | kebab-case files, folders by layer (`components/`, `hooks/`, `services/`) |
| Testing | Every feature and fix ships with tests in `tests/` |
| Git | Conventional Commits, `type/short-desc` branches, no commit unless asked |
| TypeScript | Arrow functions, named exports, no `any` / `as` / `!`, Zod at the edges |
| Tailwind | Canonical classes only, checked and fixed with `tailwindcss canonicalize` |

## Install

### Any agent

```sh
npx skills add luxuling/agent-skill-dev -g
```

Uses the [skills](https://skills.sh) CLI and works with Claude Code, Codex, Gemini CLI, Cursor, and more. Drop `-g` to install into the current project only.

### Claude Code plugin

```text
/plugin marketplace add luxuling/agent-skill-dev
/plugin install code-standards@code-standards
```

### Manual

```sh
git clone https://github.com/luxuling/agent-skill-dev.git
ln -s "$PWD/agent-skill-dev/skills/code-standards" ~/.claude/skills/code-standards
```

Use `~/.agents/skills/` instead of `~/.claude/skills/` for Codex, Gemini CLI, and other agents.

## Usage

Nothing to call. The agent loads the skill on its own whenever it writes, edits, reviews, or tests code, names a branch, or writes a commit message.

```text
> add a users page that fetches /api/users and shows a table
> fix the invoice total rounding bug
> write the commit message for this change
```

To check it is active in Claude Code, ask `what skills do you have?` and look for `code-standards`.

## Structure

```text
.claude-plugin/
└── marketplace.json              # Claude Code plugin listing
skills/code-standards/
├── SKILL.md                      # rules for every language
├── references/
│   ├── typescript.md             # TypeScript, React, Next.js, Node
│   └── tailwind.md               # Tailwind canonical classes
└── evals/
    └── evals.json                # test prompts
```

The agent reads a file under `references/` only when the task uses that stack, so other work does not pay for it.

## Customize

- **Change a rule:** edit `SKILL.md` or the matching file in `references/`.
- **Add a language:** create `references/<language>.md` and add one pointer line in `SKILL.md`.
- **Keep the spec limits:** `name` matches the folder name, `description` stays under 1024 characters, and `SKILL.md` stays under 500 lines.

## Test a change

Use the `skill-creator` skill. It runs each prompt in `evals/evals.json` with and without the skill and opens a viewer to compare the results. Output goes to `skills/code-standards-workspace/`, which is git-ignored.

## License

[MIT](LICENSE)
