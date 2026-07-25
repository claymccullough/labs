# labs

Personal Python space for running experiments. Expect many small, mostly
independent experiments rather than one cohesive application.

## Layout

- `prompts/` — numbered spec folders describing changes to make (see `/gen`)
- `ai_research/` — research notes backing each spec, mirroring its folder name
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

- Commit directly to `main`. This is a personal repo with no PR workflow;
  don't create a branch unless asked.
- Commit messages: a short summary line, then a paragraph explaining *why*
  the change was made, then bullets for specifics. Match the style of
  recent commits.
- Never force-push. On a non-fast-forward rejection, stop and report it.
- Never stage secrets, credentials, large binaries, or local scratch files.
  Call them out instead.
- Don't commit or push unless asked — `/cp` is the explicit trigger.

### Working style

- Verify rather than assume: check the actual state (`git remote -v`,
  file contents, command output) before making claims about it.
- When something is ambiguous in a way that changes the work, ask. When
  it's a routine judgment call, decide and say what was decided.
- Report honestly. If a step was skipped or a test failed, say so plainly
  with the output.

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
