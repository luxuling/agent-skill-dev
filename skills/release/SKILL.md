---
name: release
description: Release the current default-branch commit. Choose the next v* tag from the commits since the last tag, write a short release description, push only that tag, and publish that description as the GitHub release notes. That tag push is the deploy, per the ci-cd skill. Use when the user wants to release, ship a version, or runs /release. A version they name overrides the choice. Does not tag latest.
license: MIT
---

# Release

Running `/release`, or asking to release, is the ask to choose the next tag, write its description, and push that one tag. That tag push is the release.

Read the ci-cd skill and follow it. Read that skill's `SKILL.md` when it is not already loaded. Create the tag the way that skill does: `git tag <version>` then `git push origin <version>`.

## Version

A version is `v` plus numeric segments separated by dots, such as `v1.4.2`. Read a tag with fewer than three segments as three, filling the missing ones with `0`.

When the user names a version, use it. `1.4.2` is `v1.4.2`. When the word is not that shape, including `latest`, stop. A named version replaces the bump below. It does not change which commits the description uses.

Find the commits either way:

1. Take every local tag from `git tag -l 'v*' --sort=-v:refname` and every remote tag from `git ls-remote --tags origin 'v*'`. Keep names that are `v` plus numeric segments. Strip `refs/tags/` and a trailing `^{}`. The current version is the highest of those. When there is none, the commits are the ones on the default branch, and the version is `v0.1.0` unless the user named one. `v0.1.0` is the first tag ci-cd uses.
2. Otherwise the commits are `git log <current>..HEAD`. When that range is empty, stop. Do not tag the same commit again.
3. A commit is breaking when its subject has `!` before the colon, or its body has a line that starts with `BREAKING CHANGE`. A commit is a feature when its subject starts with `feat`.
4. When the user did not name a version, apply one bump. Breaking bumps the major and sets the rest to `0`. Otherwise a feature bumps the minor and sets the patch to `0`. Otherwise bump the patch.

`v1.4.2` becomes `v2.0.0` for a breaking commit, `v1.5.0` for a feature, and `v1.4.3` for anything else. A breaking commit wins over a feature in the same range.

## Description

Write the GitHub release notes from those commits. One bullet per change, the words after the colon, first letter capitalized, no type, no scope, no trailing period:

```text
- Add password reset
- Round invoice totals to 2 decimals
```

Include breaking commits, features, and subjects that start with `fix`. Leave other commits out while that list has at least one line. When it would be empty, write one bullet that says what the range did. When the list would pass eight lines, merge related lines until it is eight or fewer.

## Steps

1. Read the default branch with `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`. When that fails, stop and say so.
2. When `git status --porcelain` is not empty, stop.
3. When the current branch is not the default branch, stop and name the branch that is checked out.
4. Choose the version and the description above.
5. Look up that tag with `git rev-parse --verify refs/tags/<version>` and `git ls-remote --tags origin refs/tags/<version>`. A missing local tag makes `git rev-parse` exit non-zero, and a missing remote tag prints nothing. That means the tag is free: continue. When either command finds the tag, stop.
6. Create the tag and push only that tag:

```sh
git tag <version>
git push origin <version>
```

7. Publish the same description:

```sh
gh release create <version> --title "<version>" --notes "$(cat <<'EOF'
<description>
EOF
)"
```

8. Reply with the tag and the description, and say that this push is what starts the deploy workflow.

Do not run `git push --tags`. Do not push the branch. Do not use `gh release create --generate-notes`. Do not tag `latest`.

## Before you finish

The tag is the next version for these commits, or the version the user named. The notes are the short list above. Only that tag was pushed.
