# 0001 — Lefthook pre-commit gate: ruff lint + ty type check

**Status:** done
**Created:** 2026-07-25
**Executed:** 2026-07-25 — all tasks complete; two findings recorded below

## Goal

Broken or unlinted Python cannot be committed to this repo. A `git commit`
runs Ruff over the staged Python files and ty over the project; either failing
aborts the commit. Ruff's auto-fixable violations are corrected and re-staged
rather than bounced back for a manual fix.

This is a labs repo where experiments are written fast and reviewed rarely.
A gate at commit time is the only checkpoint that reliably runs — CI does not
exist here, and remembering to run linters by hand does not survive contact
with a working session.

## Context

The repo is uv-managed, Python >=3.14, currently with **no Python files at
all** (`find . -name "*.py" -not -path "./.venv/*"` returns nothing). The gate
is being installed before the code it will guard, so its first real exercise
comes later — verification below therefore uses a deliberately-bad throwaway
file.

Current dev dependencies in `pyproject.toml:8-12` are `ruff>=0.8.0` and
`pytest>=8.0.0`; `uv.lock` resolves ruff to 0.16.0. Neither lefthook nor ty
is a declared dependency today.

No Git hooks are installed: `.git/hooks/` holds only `*.sample` files, and
`core.hooksPath` is unset.

Two things exist on the machine but are *not* declared by the project —
lefthook 2.1.9 on `$PATH`, and ty 0.0.44 resolvable via `uv run`. The spec
does not rely on either; both become pinned dev dependencies so the gate is
reproducible rather than dependent on this machine's state.

`CLAUDE.md` requires uv for all Python invocation — never bare `python` or
`pip` — so hook commands are `uv run …`.

## Research

Full notes in [`ai_research/0001-precommit-gate/`](../../ai_research/0001-precommit-gate/).

- **Lefthook** — installs from PyPI (2.1.10) shipping its own binary, so it
  can be a uv dev dependency instead of a system prerequisite. `{staged_files}`
  expands to staged paths; omitting a file variable runs the tool project-wide.
  `stage_fixed: true` re-stages files a command modified, and works only on
  `pre-commit`. Hooks are **not** installed by cloning — `lefthook install`
  must be run per clone. → [`lefthook.md`](../../ai_research/0001-precommit-gate/lefthook.md)
- **ty** — latest 0.0.63, PyPI classifier `Development Status :: 4 - Beta`.
  Astral states breaking changes "including changes to diagnostics, may occur
  between any two versions". `ty check` exits 0 clean / **1 on warnings or
  above** / 2 on config or IO errors / 101 internal. Exit 1 covering warnings
  means the gate blocks on warning-level diagnostics by default.
  → [`ty-type-checker.md`](../../ai_research/0001-precommit-gate/ty-type-checker.md)
- **Unverified:** lefthook's behavior when `stage_fixed` rewrites a
  *partially staged* file (`git add -p`) is undocumented. Task 6 tests it
  rather than assuming. Exit codes were read from docs, not executed; task 5
  confirms them against a real bad file.

## Decisions

- **Lefthook over the `pre-commit` framework:** chose Lefthook because it was
  asked for, and the research supports it — the PyPI package pins the binary
  inside `uv.lock`, whereas `pre-commit` manages tool versions in its own
  environments, duplicating versions already tracked by uv. *(confirmed)*
- **Lefthook as a uv dev dependency, not the system binary:** the machine has
  2.1.9 on `$PATH`, but depending on that makes the gate unreproducible on any
  other clone. Pinning it in `pyproject.toml` keeps the whole toolchain in
  `uv.lock`. *(defaulted)*
- **ty pinned to an exact version, blocking on failure:** ty is beta and warns
  that diagnostics change between any two versions. Floating it means a routine
  `uv sync` can start blocking commits on untouched code. Pinning makes each
  upgrade a deliberate, reviewable commit. Rejected `--exit-zero` advisory mode
  as not actually a gate. *(confirmed)*
- ~~**Ruff on staged files, ty on the whole project:**~~ **superseded
  2026-07-25.** The premise — that scoping ty to staged files "would miss
  exactly the errors that matter" — turned out to be false. Testing showed
  `ty check <file>` follows imports and reports cross-file errors; only
  *downstream* breakage in unstaged callers is missed. Both jobs now take
  `{staged_files}`, so an unrelated broken file no longer blocks every commit.
  See [`ty-type-checker.md`](../../ai_research/0001-precommit-gate/ty-type-checker.md).
- **Ruff auto-fixes and re-stages** (`--fix` with `stage_fixed: true`) rather
  than reporting and blocking. Fewer interruptions on trivial violations. The
  trade-off, accepted knowingly: the commit's contents change after staging, so
  what lands is not exactly what was staged. Task 6 characterizes the partial
  staging edge case. *(confirmed — chosen over the report-only recommendation)*
  **Amended 2026-07-25:** kept, but the jobs are now `piped` with `ty-check`
  first rather than parallel. Auto-fix survives; what's gone is ruff rewriting
  files for a commit that then gets rejected.
- **`ruff format` excluded:** linting and formatting are separable, and adding
  a formatter would rewrite staged content more aggressively than the lint
  fixes already accepted. Can be added later. *(confirmed via scope choice)*
- **Hook commands use `uv run`:** required by `CLAUDE.md`, and guarantees the
  locked tool versions run rather than anything ambient on `$PATH`. *(defaulted)*
- **`lefthook.yml` at repo root:** the most conventional of the accepted
  filenames; `.config/lefthook.yml` is equally valid but less discoverable.
  *(defaulted)*

## Files

### Read
- `pyproject.toml` — dev dependency group the new pins are added to
- `CLAUDE.md` — records repo conventions; gains a section on the gate
- `README.md` — setup instructions a new clone follows
- `.gitignore` — needs a `lefthook-local.yml` entry

### Write
- `lefthook.yml` — **new.** Root config: a `pre-commit` hook with parallel
  `ruff-lint` and `ty-check` jobs, per the shape in Task 3.
- `pyproject.toml` — add `lefthook>=2.1.10` and `ty==0.0.63` to
  `[dependency-groups].dev`
- `uv.lock` — regenerated by `uv sync`; committed
- `.gitignore` — ignore `lefthook-local.yml` (personal overrides)
- `README.md` — document that a fresh clone must run `uv run lefthook install`
- `CLAUDE.md` — record the gate, and that `--no-verify` is the escape hatch

## Tasks

1. [x] Add `lefthook>=2.1.10` and `ty==0.0.63` to the `dev` group in
   `pyproject.toml:8-12`. Run `uv sync`. Confirm `uv run lefthook version` and
   `uv run ty --version` report the expected versions — note that ty must now
   report 0.0.63, not the ambient 0.0.44.

2. [x] Add `lefthook-local.yml` to `.gitignore` under a `# Lefthook` comment,
   so personal hook overrides never get committed.

3. [x] Write `lefthook.yml` at the repo root:

   ```yaml
   pre-commit:
     parallel: true
     jobs:
       - name: ruff-lint
         glob: "*.py"
         run: uv run ruff check --fix {staged_files}
         stage_fixed: true
       - name: ty-check
         glob: "*.py"
         run: uv run ty check
   ```

   `ty-check` deliberately takes no file variable — it checks the whole
   project. Its `glob` still gates *whether* it runs, so a commit touching no
   Python skips it.

4. [x] Run `uv run lefthook install` and confirm `.git/hooks/pre-commit` now
   exists and is no longer a `.sample`.

5. [x] Verify the gate blocks. Create a throwaway file with both an unused
   import (ruff F401, auto-fixable) and a genuine type error, e.g.:

   ```python
   import os  # noqa: intentionally unused, ruff should strip it
   def add(a: int, b: int) -> int:
       return a + b
   add("not", "ints")
   ```

   `git add` it, attempt a commit, and confirm the commit is **rejected** with
   ty reporting the bad call. Separately confirm ruff's exit behavior and that
   `uv run ty check` exits 1 on this file (`echo $?`) — the research read this
   from docs without executing it.

6. [x] Characterize the partial-staging edge case flagged in **Research**. In
   a file with two unused imports, `git add -p` only the hunk containing the
   first, then commit. Record in
   `ai_research/0001-precommit-gate/lefthook.md` under **Caveats** whether
   `stage_fixed` swept the unstaged hunk into the commit. Correct the note from
   "untested" to what actually happened.

7. [x] Delete the throwaway files from tasks 5–6, then confirm a normal commit
   of clean code passes the gate.

8. [x] Document in `README.md` (setup: fresh clones run
   `uv run lefthook install`) and `CLAUDE.md` (the gate exists; ruff auto-fixes
   and re-stages; `git commit --no-verify` bypasses it deliberately).

## Verification

- [x] `uv run lefthook version` — prints >= 2.1.10
- [x] `uv run ty --version` — prints `ty 0.0.63`, confirming the pin beat the
      ambient 0.0.44
- [x] `ls -l .git/hooks/pre-commit` — exists, not `.sample`
- [x] Committing a file with a type error — **rejected**, ty names the error
- [x] `uv run ty check <bad file>; echo $?` — prints 1
- [x] Committing a file with an unused import — import stripped and re-staged,
      commit proceeds; `git show --stat HEAD` confirms the fixed version landed
- [x] `uv run ruff check .` — clean
- [x] Committing clean code — passes with both jobs green
- [x] `git commit --no-verify` on a knowingly-bad file — succeeds, confirming
      the escape hatch works
- [x] `lefthook.md` **Caveats** updated with the observed partial-staging
      behavior, replacing the "untested" note
- [x] Human check: hook output is legible and it is obvious *which* job failed

## Out of scope

- `ruff format` — deliberately excluded; linting only (see **Decisions**)
- `pre-push` hooks and running `pytest` in a hook — there are no tests yet, and
  a slow gate is a bypassed gate
- CI — no workflows exist and none are being added
- ty rule-level configuration in `pyproject.toml` — defaults until real code
  shows they need tuning
- Retrofitting existing code — there is none

## Findings from execution (2026-07-25)

- **Partial staging is safe — open question resolved.** `stage_fixed` did
  *not* sweep unstaged content into the commit. Tested with a file holding a
  staged ruff-fixable import and a separate unstaged marker line: the commit
  contained only the staged hunk, and the unstaged change survived untouched
  in the working tree. The auto-fix decision stands; the hazard it was
  weighed against does not materialize. Recorded in
  [`lefthook.md`](../../ai_research/0001-precommit-gate/lefthook.md) **Caveats**.

- ~~**A rejected commit still mutates your working files.**~~ **Fixed
  2026-07-25.** Was: `ruff-lint` and `ty-check` ran in parallel, so ruff's
  `--fix` rewrote files even when ty blocked the commit. Now `piped: true`
  with `ty-check` first — ruff reports `(skip) broken pipe` and never runs on
  a failing commit. Verified: `import os` survived a rejected commit that
  previously stripped it.

- ~~**The installed hook runs ambient lefthook, not the pinned one.**~~
  **Fixed 2026-07-25.** Was: `.git/hooks/pre-commit` probes bare `lefthook` on
  `$PATH` before the venv copy, so ambient 2.1.9 beat the pinned 2.1.10.
  Now `.lefthookrc` (referenced as `rc: ./.lefthookrc`) exports `LEFTHOOK_BIN`,
  which the hook checks first. Verified: the banner reports v2.1.10. The `./`
  prefix is required — POSIX `.` doesn't search the working directory.

- ~~**`ty-check` project-wide scope blocks unrelated commits.**~~ **Fixed
  2026-07-25.** The decision rested on a false premise; see the struck-through
  entry in **Decisions**. Both jobs now take `{staged_files}`. Verified both
  directions: a staged file misusing an unstaged module is still caught, and
  an unrelated broken file no longer blocks a clean commit.

## Open questions

- ty 0.0.63 still has not run against real code — the repo has no Python
  files, so the gate has only ever seen throwaway test files. The first
  substantive experiment may surface diagnostics that make the default
  warning-blocks-commit behavior too strict; `--exit-zero-on-warning` is the
  pressure valve if so.
- Scoping ty to staged files means **downstream** breakage is missed: change a
  signature in a staged file and its unstaged callers go stale unnoticed. A
  `pre-push` hook running whole-project `ty check` would close that gap
  without slowing every commit. Not built — deliberately out of scope here.
