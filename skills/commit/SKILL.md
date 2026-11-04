---
name: commit
description: Create a git commit for the current change. Inspect the diff, stage only the files that belong to it, and commit with the owner's Conventional Commit message from the code-standards skill. Use when the user wants to commit changes, says commit this, or runs /commit. Does not push.
license: MIT
---

# Commit

Running `/commit`, or asking to commit, is the ask to commit. This command does not push.

Read the code-standards skill and follow its Git section for the message. Read that skill's `SKILL.md` when it is not already loaded.

## Steps

1. Run `git status`, `git diff`, `git diff --staged`, and `git log -8 --oneline`.
2. When there is nothing to commit, stop and say so. Do not create an empty commit.
3. When the user passed words after `/commit`, keep them if they already follow the Git section. Otherwise rewrite them so they do.
4. Stage the paths that belong to this change. Leave unrelated dirty files unstaged and name them. Leave secret files unstaged, including `.env` and anything that holds credentials, and name them.
5. Commit with this command, and no other flags:

```sh
git commit -m "$(cat <<'EOF'
<message>
EOF
)"
```

A hook failure means the commit did not happen. Report the hook output and stop.

Do not push. Do not skip hooks. Do not amend unless the user asked to amend.

## Before you finish

The message follows the Git section of code-standards. Secret files are unstaged. The branch was not pushed.
