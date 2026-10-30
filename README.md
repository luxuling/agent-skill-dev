<div align="center">

# code-standards

**Make AI coding agents write code your way.**

Clean, readable, consistent. No manual cleanup afterwards.

![Agent Skills](https://img.shields.io/badge/Agent_Skills-spec-black)
![Claude Code](https://img.shields.io/badge/Claude_Code-ready-d97757)
![Codex](https://img.shields.io/badge/Codex-ready-10a37f)
![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-ready-4285f4)
![License: MIT](https://img.shields.io/badge/license-MIT-blue)

```sh
npx skills add luxuling/agent-skill-dev -g
```

[Install](#-install) · [Update](#-update) · [Usage](#-usage) · [Customize](#-customize)

</div>

---

## ✨ What it enforces

| | Area | Rule |
| --- | --- | --- |
| 💬 | Comments | None by default. Only `TODO`, `FIX`, `HACK`, `WARN`, `PERF`, `NOTE`, `TEST` markers |
| 🧯 | Errors | Throw, and catch only at the boundary |
| 🛡️ | Defensive code | No needless null checks, fallback defaults, or optional chaining |
| 🧩 | Abstraction | One job per function, shared on the second use, components only render |
| 📖 | Readability | Blank lines between logical steps, never one dense block |
| 🏷️ | Naming | kebab-case files, folders by layer (`components/`, `hooks/`, `services/`) |
| 🧪 | Testing | Every feature and fix ships with tests in `tests/` |
| 🌿 | Git | Conventional Commits, `type/short-desc` branches, no commit unless asked |
| 🟦 | TypeScript | Arrow functions, named exports, no `any` / `as` / `!`, Zod at the edges |
| 🐹 | Go | Wrapped errors handled at the boundary, consumer-side interfaces, table-driven tests beside the code, chi + validator + pgx/sqlc |
| 🎨 | Tailwind | Canonical classes only, checked and fixed with `tailwindcss canonicalize` |
| ✋ | Micro-interactions | Password reveal, confirm on hard-to-undo actions, pending buttons, visible outcomes, loading/empty/error |

One folder works in Claude Code, Codex, Gemini CLI, Cursor, and any other agent that supports the [Agent Skills spec](https://agentskills.io/specification).

## 📦 Install

```sh
npx skills add luxuling/agent-skill-dev -g
```

Uses the [skills](https://skills.sh) CLI. Drop `-g` to install into the current project only.

<details>
<summary><b>Claude Code plugin</b></summary>

```text
/plugin marketplace add luxuling/agent-skill-dev
/plugin install lixu@code-standards
```

</details>

<details>
<summary><b>Manual (git clone + symlink)</b></summary>

```sh
git clone https://github.com/luxuling/agent-skill-dev.git
ln -s "$PWD/agent-skill-dev/skills/code-standards" ~/.claude/skills/code-standards
ln -s "$PWD/agent-skill-dev/skills/micro-interactions" ~/.claude/skills/micro-interactions
```

Use `~/.agents/skills/` instead of `~/.claude/skills/` for Codex, Gemini CLI, and other agents.

</details>

## 🔄 Update

| Installed with | Run |
| --- | --- |
| skills CLI | `npx skills update -g` (`npx skills check -g` to see what changed, drop `-g` for a project install) |
| Claude Code plugin | `/plugin marketplace update code-standards` then `/plugin update lixu@code-standards`, or enable auto-update in `/plugin` → Marketplaces |
| Manual | `git pull` in the clone. The symlink picks it up right away |

## 🚀 Usage

Nothing to call. The agent loads the skill on its own whenever it writes, edits, reviews, or tests code, names a branch, or writes a commit message.

```text
> add a users page that fetches /api/users and shows a table
> fix the invoice total rounding bug
> add a login form with email and password
```

To check it is active, ask `what skills do you have?` and look for `lixu:code-standards` and `lixu:micro-interactions` (plugin) or `code-standards` and `micro-interactions` (other installs).

## 🔧 Customize

- **Change a rule:** edit `SKILL.md` or the matching file in `references/`.
- **Add a language:** create `references/<language>.md` and add one pointer line in `SKILL.md`.
- **Keep the spec limits:** `name` matches the folder name, `description` stays under 1024 characters, `SKILL.md` stays under 500 lines.
- **Test a change:** run the `skill-creator` skill. It runs each prompt in `evals/evals.json` with and without the skill and opens a viewer to compare. Output goes to `skills/code-standards-workspace/`, which is git-ignored.

<details>
<summary><b>📁 Structure</b></summary>

```text
.claude-plugin/
└── marketplace.json              # Claude Code plugin listing
skills/code-standards/
├── SKILL.md                      # rules for every language
├── references/
│   ├── typescript.md             # TypeScript, React, Next.js, Node
│   ├── go.md                     # Go
│   └── tailwind.md               # Tailwind canonical classes
└── evals/
    └── evals.json                # test prompts
skills/micro-interactions/
├── SKILL.md                      # password reveal, confirms, feedback
└── evals/
    └── evals.json                # test prompts
```

Files under `references/` load only when the task uses that stack.

</details>

## License

[MIT](LICENSE)
