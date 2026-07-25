# labs

Personal Python space for running experiments. Expect many small, mostly
independent experiments rather than one cohesive application.

## Layout

- `prompts/` — numbered spec folders describing changes to make (see `/gen`)
- `ai_research/` — research notes backing each spec, mirroring its folder name
- `decisions/` — timestamped log of every decision Clay has made
- `data/` — local experiment data; gitignored except `.gitkeep`
- `docs/` — notes and writeups
- `.claude/commands/` — project slash commands

## Tooling

This project uses [uv](https://docs.astral.sh/uv/). Never call `pip` or
`python` directly.

```bash
uv sync              # set up the environment
uv add <package>     # add a dependency
uv run <script>.py   # run a script
uv run ruff check .  # lint
uv run pytest        # test
```

## Conventions

These are the working preferences observed in practice. Correct anything
here that stops matching how you want to work.

### Git

- Commit directly to `main` by default. This is a personal repo with no PR
  workflow; don't create a branch unless asked. Running `/pr` counts as
  asking — it branches, moves the work over, and leaves `main` clean.
- Commit messages: a short summary line, then a paragraph explaining *why*
  the change was made, then bullets for specifics. Match the style of
  recent commits.
- Never force-push. On a non-fast-forward rejection, stop and report it.
- Never stage secrets, credentials, large binaries, or local scratch files.
  Call them out instead.
- Don't commit or push unless asked — `/cp` is the explicit trigger.

### Pre-commit gate

A lefthook pre-commit hook runs on every commit, configured in `lefthook.yml`.
Fresh clones must run `uv run lefthook install` — cloning a repo does not
install its hooks.

Two jobs run **sequentially** (`piped: true`), stopping at the first failure:

1. `ty-check` — `ty check` on the staged Python files.
2. `ruff-lint` — lints those files, auto-fixing with `--fix` and re-staging
   the fixes via `stage_fixed`.

The order is deliberate. Ruff runs with `--fix`, so if it ran alongside a
failing type check it would rewrite files on disk for a commit that then gets
rejected. Types first means **a rejected commit leaves your working tree
untouched**.

Both jobs pass `{staged_files}`, but ty still follows imports out of them — a
staged file that misuses something from an unstaged module is caught. What it
won't catch is the reverse: changing a signature in a staged file while its
callers, unstaged, go stale. Run `uv run ty check` yourself for a whole-project
sweep.

`git commit --no-verify` bypasses the gate deliberately — use it as the
escape hatch when you need to commit through a failure.

`.lefthookrc` pins the hook to the lefthook in `uv.lock`. Without it the
generated hook probes bare `lefthook` on `$PATH` first and would run whatever
version happens to be installed system-wide.

### Working style

- Verify rather than assume: check the actual state (`git remote -v`,
  file contents, command output) before making claims about it.
- When something is ambiguous in a way that changes the work, ask. When
  it's a routine judgment call, decide and say what was decided.
- Report honestly. If a step was skipped or a test failed, say so plainly
  with the output.
- Separate the agent doing the work from the agent checking it. Whoever
  wrote the code is the wrong one to grade it.

### Decisions

Every explicit decision Clay makes gets logged to `decisions/` — one
timestamped file per decision, recording the choice, the alternatives
rejected, his reasoning, and the context. See `decisions/README.md`.

The point is to learn his preferences, so:

- **Read `decisions/` before asking anything.** A logged preference is a
  default to apply, not a question to re-ask. Say when you're applying one.
- Log corrections and overrides of your recommendation with particular
  care — disagreement carries more signal than agreement.
- Record his stated reasoning, or note that he didn't give one. Never
  invent a rationale to fill the gap.
- When a new decision contradicts an old one, the new one wins; mark the
  old **superseded** rather than deleting it.

This does not override surfacing genuinely new design choices — the line is:
surface what's new, apply what's settled.

### Python defaults

General conventions for this repo, not preferences you've stated. Change
them freely.

- Format and lint with `ruff`; test with `pytest`.
- Prefer standard library and small, well-scoped dependencies.
- Experiments can be single scripts. Don't build package scaffolding,
  abstractions, or config layers an experiment doesn't need.
- Write experiment data to `data/`; it's gitignored, so it's safe for
  large or messy intermediate files.
- Keep the notes for an experiment near the experiment.

## Specs

`/gen <description>` writes a spec to `prompts/NNNN-slug/spec.md`.
`/exe <NNNN|slug>` executes one. A spec states its goal, the decisions it
makes, the files to read and write, the tasks to perform, and how to verify
the result.

`/exe` does not implement directly: it dispatches each task to a fresh
**Sonnet** subagent, one at a time in order, then runs the spec's
**Verification** checks itself. Subagents carry none of the orchestrator's
context, so each prompt has to stand alone — spec text, relevant decisions
and research, files in scope, conventions, and what earlier tasks changed.
Their reports are evidence, not proof; check the files they claim to have
changed.

Before writing, `/gen` researches anything external the change touches —
libraries, algorithms, APIs, formats — against current sources rather than
from memory. Library and approach choices should rest on what that turned
up, not on recollection.

Research is **saved to `ai_research/NNNN-slug/`**, mirroring the spec folder
name, and committed. One markdown file per topic, holding findings, verbatim
excerpts of what drove a decision, options considered, and caveats about what
may go stale. This is a durable knowledge base, not scratch output: `/gen`
checks it before researching and reuses what's there (rechecking currency
first), and `/exe` reads it for the API shapes and constraints the spec
assumed. When implementation shows a note was wrong, the note gets corrected
along with the code.

Specs are iterated, not written once. `/gen` surfaces architectural and
design decisions for a call *before* writing — it should never bury a
consequential choice by picking silently. Running `/gen` again with feedback
revises the spec in hand rather than starting a new one; a new index is for
a genuinely separate change.

Every decision is recorded in the spec's **Decisions** section, marked
*confirmed* (you chose it) or *defaulted* (I picked a reasonable default).
`/exe` may challenge a *defaulted* decision if the code contradicts it, but
won't relitigate a *confirmed* one.
