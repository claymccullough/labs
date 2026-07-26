# Research gets saved to a committed knowledge base, not thrown away

**When:** 2026-07-25 12:50 CDT
**Context:** Designing the `/gen` command — after asking that it research libraries and algorithms before speccing, he asked that the findings be persisted
**Status:** active

## Choice

Research output is written to `ai_research/NNNN-slug/`, mirroring the spec
folder name, and committed. One markdown file per topic, holding findings,
verbatim excerpts of what drove a decision, options compared, and caveats.
`/gen` checks it before researching and reuses what's there; `/exe` reads it.

Structure chosen: top-level `ai_research/` rather than nested inside each
spec folder, with notes plus key excerpts rather than links-only or full
page captures.

## Alternatives rejected

- Nesting research inside `prompts/NNNN-slug/` — rejected; scatters it and
  makes reuse across specs harder
- Links and short notes only — rejected; evidence dies with the source
- Full page captures — rejected as too noisy for the repo

## Reasoning

Stated goal was that research be "remembered for later." The consistent
thread across his requests: work products should accumulate rather than
evaporate at the end of a session.

## Generalizes to

Strong signal, and it repeats — see also [decisions logging](2026-07-25T1320-log-every-decision.md).
He wants durable artifacts over ephemeral output. When there is a choice
between doing work in-conversation and writing it somewhere durable, default
to writing it down.

Corollary he asked for explicitly: reuse before redoing, but recheck currency
first rather than trusting a stale note.
