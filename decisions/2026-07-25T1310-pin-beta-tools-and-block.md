# Pin beta tooling to an exact version and let it block commits

**When:** 2026-07-25 13:10 CDT
**Context:** [0001-precommit-gate](../prompts/0001-precommit-gate/spec.md) — choosing how the pre-commit gate should handle `ty`, which is beta at 0.0.63 and warns that diagnostics change between any two versions
**Status:** active

## Choice

Pin `ty` to an exact version (`ty==0.0.63`) and let type errors block the
commit. Upgrades become a deliberate, reviewable change.

## Alternatives rejected

- Pin but run `--exit-zero` (advisory only) — rejected; that is not a gate
- Float `ty>=0.0.63` and block — rejected; a routine `uv sync` could start
  blocking commits on untouched code
- Skip `ty` until it hits 1.0 — rejected; he wanted the type checking

## Reasoning

Not stated directly — he selected the recommended option after being shown
that unpinned beta tooling can break commits on code the author never
touched. The choice is consistent with wanting a gate that actually blocks
while refusing to let it fire unpredictably.

## Generalizes to

Likely a repo-wide default for pre-1.0 dependencies: pin exactly, upgrade
deliberately. Worth treating as the default for any beta tool in a blocking
position until he says otherwise. Weaker signal for stable 1.x+ dependencies,
where floating is more normal.
