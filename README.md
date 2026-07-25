# labs

Personal Python space for running experiments.

## Setup

This project uses [uv](https://docs.astral.sh/uv/) for dependency and environment management.

```bash
uv sync
```

## Layout

- `data/` — local experiment data (gitignored, not committed)
- `docs/` — notes and writeups for experiments

## Usage

Add dependencies as needed:

```bash
uv add <package>
```

Run a script:

```bash
uv run <script>.py
```
