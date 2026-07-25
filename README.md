# labs

Personal Python space for running experiments.

## Setup

This project uses [uv](https://docs.astral.sh/uv/) for dependency and environment management.

```bash
uv sync
```

## Layout

- `prompts/` — numbered specs describing changes to make
- `ai_research/` — research notes backing each spec, by spec folder name
- `data/` — local experiment data (gitignored, not committed)
- `docs/` — notes and writeups for experiments

## Commands

Project slash commands, available in Claude Code:

- `/gen <description>` — write a spec to `prompts/NNNN-slug/spec.md`
- `/exe <NNNN|slug>` — execute a spec and run its verification steps
- `/cp` — commit and push

## Usage

Add dependencies as needed:

```bash
uv add <package>
```

Run a script:

```bash
uv run <script>.py
```
