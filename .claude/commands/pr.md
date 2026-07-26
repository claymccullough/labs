---
description: Branch if needed, commit, push, and open a GitHub PR
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*), Bash(git branch:*), Bash(git switch:*), Bash(git checkout:*), Bash(git remote:*), Bash(git ls-remote:*), Bash(git rev-parse:*), Bash(git rev-list:*), Bash(git merge-base:*), Bash(git reset:*), Bash(git stash:*), Bash(gh pr:*), Bash(gh repo:*), Bash(gh auth:*), Read, AskUserQuestion
---

Get the current work onto a branch and open a pull request for it.

Optional PR title (may be empty — generate one if so): $ARGUMENTS

## Context

- Branch: !`git branch --show-current`
- Status: !`git status --short`
- Unpushed commits: !`git log --oneline @{u}.. 2>/dev/null || echo "(no upstream set)"`
- Recent commits (match this message style): !`git log --oneline -10`
- Remote: !`git remote -v | head -1`

## Steps

1. **Check there's something to PR.** If the working tree is clean and there
   are no unpushed commits, say so and stop.

2. **Get onto a branch.** Per repo convention Clay works directly on `main`,
   so this usually needs a new one.

   If already on a non-default branch, use it as-is. Otherwise:
   - Pick a short kebab-case name from the actual change — `add-ruff-gate`,
     not `changes`. Use `$ARGUMENTS` as the source if given.
   - **Capture the upstream ref before switching** — `@{u}` resolves against
     whatever branch is current, so reading it after the switch fails:

     ```bash
     UP=$(git rev-parse --abbrev-ref main@{u})   # e.g. origin/main
     git switch -c <name>                        # uncommitted work follows
     git branch -f main "$UP"                    # only if main had unpushed commits
     ```

     `git branch -f` moves a ref that isn't checked out. The commits stay on
     the new branch, `main` returns to matching its upstream, and the working
     tree is untouched. Verified: two unpushed commits and an uncommitted file
     all survived this exact sequence.
   - Skip the `branch -f` step entirely if `main` had no unpushed commits
     (`git rev-list --count @{u}..HEAD` is 0). Nothing to move.
   - **Never `git reset --hard`, and never force-push.** If the situation
     needs either, stop and explain instead.

3. **Commit anything uncommitted.** Review the diff first. Stage deliberately —
   never `git add -A` blindly. Don't stage secrets, credentials, large
   binaries, or local scratch files; call those out instead. Write the message
   in the repo's style: summary line, a paragraph on *why*, then bullets.

4. **Push** with `git push -u origin HEAD`. If it's rejected as
   non-fast-forward, stop and report — do not force.

5. **Open the PR** with `gh pr create`. Generate the title and body from the
   actual commits and diff, not from a guess:
   - Title: `$ARGUMENTS` if given, else a concise summary of the change.
   - Body: what changed and why, plus a test plan covering how it was
     verified. Note anything left undone or needing a human eye.
   - Base the PR on the repo's default branch.
   - Pass the body via `--body-file -` and a heredoc so formatting survives.

6. **Report the PR URL.** Also say what branch was created and, if commits were
   moved off `main`, confirm `main` now matches its upstream.

7. If Clay made any decision during this — a branch name, a scope call, an
   override — record it in `decisions/` per `decisions/README.md`.

## Conventions

- Read `CLAUDE.md` and follow it.
- Check `decisions/` before asking. A logged preference is a default to apply.
- The pre-commit gate runs on commit here. If it rejects the commit, fix the
  problem rather than reaching for `--no-verify`; if you do bypass it, say so
  explicitly in the report and in the PR body.
- Never force-push, never `reset --hard`, never rewrite pushed history.
- End the PR body with:

  ```
  🤖 Generated with [Claude Code](https://claude.com/claude-code)
  ```
