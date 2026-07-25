# Record every explicit decision to a timestamped log, to learn preferences

**When:** 2026-07-25 13:20 CDT
**Context:** After several rounds of building the spec workflow, where he repeatedly made choices via AskUserQuestion
**Status:** active

## Choice

A top-level `decisions/` directory, one file per decision, timestamp-prefixed
(`YYYY-MM-DDTHHMM-slug.md`), recording the choice, alternatives rejected, his
reasoning, and the context it was made in.

Scope: **any** explicit decision — AskUserQuestion answers, corrections,
preferences stated in passing — not only spec-related ones.

## Alternatives rejected

- Spec-related decisions only — rejected as too narrow
- Nested under each spec folder — rejected; scatters the record

## Reasoning

Stated directly: "That way you can learn my prefs." The purpose is not an
audit trail but a working model of how he decides, so the same question does
not get asked twice.

## Generalizes to

A standing instruction, not a one-off. It changes default behavior everywhere:
check `decisions/` before asking, and treat a logged preference as a default
to apply rather than a fact to re-confirm.

Implies a bar for asking at all — if he has to answer the same class of
question repeatedly, that is a failure to have learned. Balanced against his
equally explicit instruction that consequential design choices be surfaced
rather than assumed, the line is: surface genuinely new choices, apply
already-settled ones.

See [research persistence](2026-07-25T1250-research-persisted-as-knowledge-base.md)
for the same accumulate-don't-evaporate instinct.
