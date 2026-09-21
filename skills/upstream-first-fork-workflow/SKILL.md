---
name: upstream-first-fork-workflow
description: Manage a fast-moving upstream fork with local bug fixes or features. Use when comparing, rebasing, syncing, or retiring fork-only changes.
---

# Upstream-First Fork Workflow

`upstream/main` is authoritative for behavior shared with the original project.
Use a worktree and follow the repository's `AGENTS.md`.

## Branch Roles

- `upstream/main`: the original project. Never modify it.
- `origin/main`: a clean mirror branch. Update it only with upstream-only history.
- `origin/<feature>`: fork-only bug fixes and features. Rebase it onto `upstream/main`.

Do not mix fork-only commits into the mirror branch. Do not merge upstream into a
feature branch when rebase can preserve a linear history.

## Before Changing Code

1. Fetch both remotes and record the target upstream commit.
2. Review the local diff and the upstream changes for the same bug report or
   requirement. Compare observable behavior and relevant tests, not filenames
   or commit messages alone.
3. Work in a branch worktree. Keep unrelated local edits untouched.

## Upstream Wins

When upstream fully solves the same problem:

1. Keep upstream's implementation and tests.
2. Delete the overlapping fork-only code, tests, docs, and configuration.
3. Drop the duplicate local commit during rebase, or amend it so it contains
   only behavior that upstream does not provide.

Do not keep duplicate fallbacks, alternate implementations, or compatibility
switches merely because they were previously developed in the fork.

## Fork Delta

Keep a fork-only change only when upstream does not solve the requirement, or
when a verified, user-visible gap remains. State that gap in the commit message
and add the smallest targeted test.

If upstream partially overlaps, reduce the local change to the uncovered
behavior before committing. Re-check the resulting diff against `upstream/main`.

## Sync Procedure

For a feature branch:

```powershell
git fetch origin upstream
git rebase upstream/main
git diff upstream/main...HEAD
git push --force-with-lease origin <feature-branch>
```

For a pure upstream mirror branch, push only the upstream commit range. Never
force-push a shared branch unless its owners have agreed to that history change.

After syncing, report the upstream commit, the fork branch, retained fork-only
files, and discarded overlapping files. Verify the branch has no uncommitted
changes and run focused checks for the retained delta.
