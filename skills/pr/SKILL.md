---
name: pr
description: Open a pull request for the current branch. Push only that branch and create the PR with gh, with a Conventional Commit title from the code-standards skill. Use when the user wants to open a pull request or runs /pr. Does not merge.
license: MIT
---

# Pull request

Running `/pr`, or asking to open a pull request, is the ask to push this branch and open the PR. This command does not merge, and it does not commit.

Read the code-standards skill and follow its Git section for the branch name and the PR title. Read that skill's `SKILL.md` when it is not already loaded.

## Steps

1. Read the default branch with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`. When that fails, stop and say so.
2. When the current branch is that default branch, stop. A PR needs a branch named the way the Git section says. Create one only when the user names it in this request, and only when that name follows the Git section.
3. When `git status --porcelain` is not empty, stop. Tell the user to commit first with `/commit`.
4. When `gh pr view --json url --jq .url` already returns a PR, give that URL and stop.
5. Push only this branch: `git push -u origin HEAD`.
6. Open the PR with `gh pr create`. The base is the default branch. The title follows the Git section. The body says what changed and why, taken from the commits on this branch:

```sh
gh pr create --base <default> --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

7. Give the PR URL.

Do not merge. Do not push any other branch or any tag.

## Before you finish

The branch name and title follow the Git section of code-standards. Only this branch was pushed. The PR is open and not merged.
