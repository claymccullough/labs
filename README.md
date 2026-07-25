# labs

Personal Python space for running experiments.

## Setup

This project uses [uv](https://docs.astral.sh/uv/) for dependency and environment management.

```bash
uv sync
```

Cloning does not install the git hooks — each clone must run this once:

```bash
uv run lefthook install
```

## Layout

- `prompts/` — numbered specs describing changes to make
- `ai_research/` — research notes backing each spec, by spec folder name
- `decisions/` — timestamped log of decisions made along the way
- `data/` — local experiment data (gitignored, not committed)
- `docs/` — notes and writeups for experiments

## Commands

Project slash commands, available in Claude Code:

- `/gen <description>` — research, surface design decisions, then write a
  spec to `prompts/NNNN-slug/spec.md`
- `/exe <NNNN|slug>` — execute a spec via Sonnet subagents, one task at a
  time, then verify
- `/cp` — commit and push to the current branch
- `/pr [title]` — branch if needed, commit, push, and open a GitHub PR

## Usage

Add dependencies as needed:

```bash
uv add <package>
```

Run a script:

```bash
uv run <script>.py
```
