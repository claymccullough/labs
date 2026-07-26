# Fix all three gate sharp edges, including reversing two earlier decisions

**When:** 2026-07-25 14:10 CDT
**Context:** After [0001-precommit-gate](../prompts/0001-precommit-gate/spec.md) shipped in PR #1, which flagged three known rough edges
**Status:** active

## Choice

Fix all three, rather than accepting any as inherent trade-offs:

1. Hook pinned to `uv.lock`'s lefthook via `LEFTHOOK_BIN` in `.lefthookrc`.
2. Jobs `piped` with `ty-check` first, so a rejected commit no longer leaves
   ruff's rewrites on disk.
3. `ty check` scoped to `{staged_files}`, so an unrelated broken file no longer
   blocks every commit.

Two of these reverse decisions from the original spec.

## Alternatives rejected

- Leaving the ambient/pinned lefthook gap as-is — it was logged as an open
  question with the reasoning that the version gap was trivial. He fixed it anyway.
- Accepting file mutation as the price of auto-fix — turned out to be a false
  choice; `piped` preserves auto-fix and removes the mutation.
- Accepting project-wide ty as the price of cross-file checking — also a false
  choice; scoped ty still follows imports.

## Reasoning

Not stated — the instruction was "Fix these things," and he selected all three
from the list. Notably he did *not* select the unrelated shell noise, so the
scope was the gate specifically.

Worth recording that two of these were presented to him as trade-offs he had
already accepted, and he chose to revisit rather than live with them. The
research that followed showed both "trade-offs" rested on false premises: the
mutation was an artifact of parallel execution, and the scope limitation was an
assumption about ty never tested.

## Generalizes to

A flagged rough edge is not a settled one. When something is surfaced as a
known limitation, the expectation is that it gets fixed unless it's genuinely
impossible — not filed as a permanent caveat.

Corollary for how to work: a stated trade-off deserves a test before being
accepted. Both reversals here came from running a five-line experiment against
an assumption that had been carried since the spec was written. Same instinct as
[research before speccing](2026-07-25T1250-research-persisted-as-knowledge-base.md)
— verify rather than reason from what seems obvious.
