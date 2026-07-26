# Decisions

A running record of choices Clay has made — what was chosen, what was
rejected, and why. The point is accumulated preference: any one entry is a
footnote, but read together they should make it possible to anticipate a
call instead of asking again.

## Filename

`YYYY-MM-DDTHHMM-short-slug.md`, e.g. `2026-07-25T1324-ty-version-pinning.md`.
Local time, chronological when sorted. One decision per file — resist the
urge to bundle related choices, since they get superseded independently.

## What gets recorded

Any decision Clay makes explicitly:

- Answers to `AskUserQuestion` prompts
- Corrections — "no, do it this way", overriding a recommendation
- Preferences stated in passing ("always use uv", "don't commit unless I ask")
- Choices between approaches during a conversation
- Reversals of an earlier decision — these matter most; link the superseded entry

Not recorded: things I decided myself with no input, or restatements of an
already-logged preference with nothing new in them.

## Reading these

Before asking, check whether it's already been answered here. A logged
preference is a default to apply, not a fact to re-confirm.

Two cautions. A decision recorded in one context may not generalize to
another — the entry's **Context** section is what tells you the difference.
And these age: a preference from a repo's first week may not survive contact
with its actual use. When an entry conflicts with what Clay says now, what
he says now wins, and the old entry gets superseded rather than deleted.

## Template

```markdown
# <Decision in one line>

**When:** YYYY-MM-DD HH:MM TZ
**Context:** <what was being worked on — link the spec or file if relevant>
**Status:** active | superseded by [<slug>](<file>.md)

## Choice

What Clay chose, stated plainly.

## Alternatives rejected

What else was on the table, and — where he said — why it lost.

## Reasoning

His stated reasoning, in his words where possible. If he did not give one,
say so rather than inventing a rationale. An unexplained preference is still
a preference; a fabricated justification is worse than a blank.

## Generalizes to

How far this reaches: a one-off for this task, a repo-wide default, or a
standing preference. Say if unclear rather than over-claiming.
```
