# /exe implements via Sonnet subagents, one per task, verified by the orchestrator

**When:** 2026-07-25 13:22 CDT
**Context:** Revising the `/exe` command after the spec workflow was in place
**Status:** active

## Choice

`/exe` does not implement directly. It delegates each spec task to a fresh
Sonnet subagent, sequentially so later tasks see earlier results, then runs
the spec's **Verification** commands itself rather than trusting self-reports.

Stated requirement: "always execute using sonnet subagents only."

## Alternatives rejected

- Parallel fan-out across independent tasks — rejected; risks conflicting
  edits to shared files
- One subagent for the whole spec end to end — rejected; less checkpointing
- Subagents verifying their own work — rejected in favor of independent
  verification by the orchestrator

## Reasoning

The model choice was stated as a requirement without elaboration — most
plausibly cost or speed, since Sonnet is cheaper than Opus for mechanical
implementation work against an already-written spec.

The verification choice he did explain by selecting it: the same agent
grading its own work misses things.

## Generalizes to

Model selection: prefer Sonnet for well-specified implementation work.
Reserve the expensive model for the parts where judgment matters — writing
the spec, surfacing decisions, verifying results.

The deeper pattern, which shows up repeatedly: separate the agent doing the
work from the agent checking it. Same instinct as
[auto-fix but verify](2026-07-25T1311-ruff-autofix-and-restage.md) — willing
to automate execution, unwilling to skip the check.
