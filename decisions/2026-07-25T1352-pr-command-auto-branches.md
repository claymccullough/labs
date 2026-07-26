# /pr auto-branches off main and moves unpushed work onto the branch

**When:** 2026-07-25 13:52 CDT
**Context:** Adding a `/pr` slash command, which conflicts with the standing convention of committing directly to `main` — a PR needs a branch
**Status:** active

## Choice

`/pr` creates a branch automatically when invoked from `main`, moves both
uncommitted changes and any unpushed `main` commits onto it, commits, pushes,
and opens the PR with a generated title and body. `main` is left matching its
upstream.

## Alternatives rejected

- Prompt for a branch name each time — rejected as friction on every run
- Refuse to run on `main` — rejected; useless in his normal working state
- Leave unpushed `main` commits alone and PR only uncommitted work — rejected;
  splits one piece of work across two places
- Draft PRs by default, or title-only with no body — both rejected

## Reasoning

Not stated; he took the recommended option on all three questions. The choices
are consistent with a preference visible elsewhere: automate the mechanical
parts, keep the safety checks. He wanted the command to work from where he
actually is rather than demanding he prepare the repo first.

## Generalizes to

Refines rather than overturns [commit directly to main]. The convention holds
by default; `/pr` is the explicit opt-in when a change warrants review, and
"don't create a branch unless asked" is satisfied because invoking `/pr` *is*
the asking.

Same automate-but-verify instinct as
[ruff auto-fix](2026-07-25T1311-ruff-autofix-and-restage.md) and
[exe delegating to subagents](2026-07-25T1322-exe-delegates-to-sonnet-subagents.md):
he accepts automation doing consequential things, provided the destructive
operations stay off the table. Hence the command's hard prohibition on
`reset --hard` and force-push.
