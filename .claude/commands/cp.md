---
description: Commit all changes and push to the remote
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git branch:*), Bash(git remote:*), Bash(git ls-remote:*)
---

Commit the current changes and push them.

Extra instruction from the user (may be empty; if it contains a commit message, use it): $ARGUMENTS

## Context

- Status: !`git status`
- Staged diff: !`git diff --cached --stat`
- Unstaged diff: !`git diff --stat`
- Recent commits (match this message style): !`git log --oneline -10`

## Steps

1. Review the changes above. If there is nothing to commit, say so and stop.
2. Inspect anything you are unsure about with `git diff` before staging.
3. Stage the changes with `git add`. Do not stage files that look like
   secrets, credentials, large binaries, or local-only scratch files —
   call those out instead of committing them.
4. Commit with a message describing *why* the change was made, following
   the style of the recent commits above. Use `$ARGUMENTS` as the message
   if the user supplied one.
5. Push to the tracked remote. If the branch has no upstream, use
   `git push -u origin HEAD`.
6. If the push is rejected as non-fast-forward, stop and report it. Do not
   force-push.
7. Report the resulting commit SHA and what was pushed.
