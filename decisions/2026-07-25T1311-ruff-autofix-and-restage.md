# Let the pre-commit hook auto-fix and re-stage, rather than report and block

**When:** 2026-07-25 13:11 CDT
**Context:** [0001-precommit-gate](../prompts/0001-precommit-gate/spec.md) — whether the Ruff pre-commit job should fix violations automatically or just report them
**Status:** active

## Choice

Run `ruff check --fix` with `stage_fixed: true`, so auto-fixable violations
are corrected and re-staged into the commit automatically.

**This overrode my recommendation**, which was report-only.

## Alternatives rejected

- Report only, block, fix by hand (my recommendation) — rejected
- Add `ruff format` alongside, also auto-staged — rejected as too aggressive

## Reasoning

Not stated. He was shown the trade-off explicitly — that auto-fixing changes
the commit's contents after staging, so what lands is not what was reviewed,
with an undocumented risk around partially staged files — and chose it anyway.

The reading: for a solo experiments repo, fewer interruptions on trivial
violations beats strict what-you-staged-is-what-you-commit fidelity. He is
willing to trade a small correctness guarantee for flow.

## Generalizes to

Tooling ergonomics in this repo: prefer automation that removes friction over
automation that stops to ask, when the blast radius is small and it's his own
work. Do **not** extend this to shared repos, or to anything where a silent
rewrite could reach someone else's work.

Note the boundary — he rejected `ruff format` in the same breath, so this is
not blanket "auto-fix everything". Lint fixes yes, wholesale reformatting no.
