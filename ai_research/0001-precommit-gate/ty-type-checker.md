# ty (Astral type checker)

**Researched:** 2026-07-25 · **For:** [0001-precommit-gate](../../prompts/0001-precommit-gate/spec.md)
**Sources:** https://pypi.org/pypi/ty/json · https://docs.astral.sh/ty/ · https://docs.astral.sh/ty/reference/exit-codes/

## Question

Is ty ready to gate commits on, how is it invoked, and what do its exit codes
mean for a hook that blocks on failure?

## Findings

ty is "an extremely fast Python type checker, written in Rust," from Astral
(the uv and Ruff people). Claims 10x–100x faster than mypy/Pyright. Ships a
language server for IDE integration.

**Version and stability (2026-07-25):** latest is **0.0.63**. PyPI classifier
is `Development Status :: 4 - Beta`. The project states it is in beta and
that `0.0.x` versioning means **no stable API — breaking changes, including
changes to diagnostics, may occur between any two versions.**

This is the central constraint for a commit gate. An unpinned ty can change
what it flags after a routine `uv sync`, and start blocking commits on code
that was fine the day before and that the author did not touch.

Locally, `uv run ty --version` reports **0.0.44** — older than PyPI's 0.0.63,
resolved transiently and not currently in `pyproject.toml`.

**Invocation:** `ty check` — checks all Python files in the working directory
or project by default. Configured via `pyproject.toml`; supports configurable
rule levels, per-file overrides, and suppression comments.

### Exit codes

| Code | Meaning |
| --- | --- |
| 0 | No violations with severity `warning` or higher |
| 1 | Violations with severity `warning` or higher were found |
| 2 | Invalid CLI options, invalid configuration, or IO errors |
| 101 | Internal error |

**Exit 1 includes warnings, not just errors.** A gate that blocks on non-zero
therefore blocks on warning-level diagnostics too. Flags that change this:

- `--exit-zero` — always exit 0 even with violations (advisory mode)
- `--error-on-warning` — exit 1 for any violation at warning or above
- `--exit-zero-on-warning` — exit 1 only for error-severity violations

`--error-on-warning` is mutually exclusive with `--exit-zero` and
`--exit-zero-on-warning`.

## Key excerpts

> ty does not yet have a stable API; breaking changes, including changes to
> diagnostics, may occur between any two versions.
> — https://pypi.org/pypi/ty/json

> An extremely fast Python type checker, written in Rust.
> — https://pypi.org/pypi/ty/json

> `0` if no violations with severity `warning` or higher were found
> `1` if violations with severity `warning` or higher were found
> `2` if invalid CLI options, invalid configuration, or IO errors
> `101` internal error
> — https://docs.astral.sh/ty/reference/exit-codes/

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| Pin exact (`ty==0.0.63`), block | Deterministic; upgrades are a deliberate commit | Manual bumps; misses new checks until bumped |
| Floating (`ty>=0.0.63`), block | Always newest diagnostics | A sync can block commits on untouched code — bad for a gate |
| Pin, `--exit-zero` (advisory) | Never blocks | Not a gate; warnings get ignored in practice |
| mypy instead | Stable, mature, 1.x | Much slower; separate config; outside the Astral toolchain already in use |
| Pyright instead | Mature, excellent inference | Node dependency in a uv project |

## Conclusion

Adopt ty, pinned to an exact version (`ty==0.0.63`), blocking the commit on
non-zero exit. Pinning is what makes beta software safe to gate on: upgrades
become an explicit, reviewable change rather than a surprise. Default exit
behavior is kept (no `--exit-zero`), so warnings block too — appropriate for a
repo starting with zero Python files and therefore zero legacy violations.
See the spec's **Decisions**.

## Follow-up finding (2026-07-25): scoping does not lose cross-file checking

The original spec ran `ty check` with no arguments (whole project) on the
premise that scoping it to staged files would miss cross-file errors. **That
premise was wrong**, and the docs are ambiguous on it, so it was tested directly.

Setup: `lib.py` defines `add(a: int, b: int) -> int`; `caller.py` imports it and
calls `add("wrong", "types")`.

| Command | Result |
| --- | --- |
| `ty check caller.py` | **Catches the error** — follows the import into `lib.py` |
| `ty check lib.py` | Passes — ignores the broken caller |

So `ty check {staged_files}` still resolves imports out of the staged files and
reports type errors across that boundary. What it does *not* do is check
**downstream** — editing `lib.py`'s signature while an unstaged `caller.py` goes
stale is missed, confirmed by the second row.

That's the real trade, and it's much narrower than "no cross-file checking":
scoped ty catches what a staged file gets wrong about the rest of the codebase,
but not what the rest of the codebase gets wrong about a staged file. In
exchange, a type error in an unrelated file no longer blocks every commit.

**The downstream half is covered at push time.** A `pre-push` job runs
`ty check` with no file argument (whole project), so the two hooks together
leave no gap: scoped-and-fast on commit, exhaustive before anything leaves the
machine. Verified by changing a signature while staging only the definition —
the commit passed, the push was rejected with `missing-argument` pointing at
the stale caller, and the push succeeded once it was fixed.

Worth knowing for anyone reusing this pattern: `ty check` with no path argument
on a repo containing no Python emits `WARN No python files found under the
given path(s)` and still exits 0, so an empty project does not block pushes.

## Caveats

- **Beta, pre-1.0.** Expect diagnostic churn on every bump. Re-read the ty
  changelog when bumping the pin; a bump may surface new violations across
  the repo at once.
- The local 0.0.44 predates the researched 0.0.63; the spec pins explicitly
  rather than inheriting whatever is resolved.
- Exit codes verified against docs, not executed here. Worth confirming
  `ty check` exit 1 on a known-bad file during implementation.
- Recheck when ty reaches 1.0 — the pinning rationale weakens once the API
  stabilizes, and the pin could relax to a compatible range.
