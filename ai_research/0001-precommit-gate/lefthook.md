# Lefthook

**Researched:** 2026-07-25 · **For:** [0001-precommit-gate](../../prompts/0001-precommit-gate/spec.md)
**Sources:** https://lefthook.dev/configuration/ · https://lefthook.dev/configuration/run/ · https://lefthook.dev/examples/stage_fixed/ · https://pypi.org/pypi/lefthook/json

## Question

How do you configure a Lefthook pre-commit hook to run linters on staged
files, and how do you install Lefthook in a uv-managed Python project?

## Findings

Lefthook is a polyglot Git hooks manager by Evil Martians — a single
dependency-free binary, written in Go, that runs commands in parallel.

**Versions (2026-07-25):** PyPI `lefthook` 2.1.10 is the latest. The machine
already has 2.1.9 installed at `/home/clay/.local/bin/lefthook`.

**Installable from PyPI.** The `lefthook` PyPI package ships the compiled
binary, so it can be a dev dependency rather than a system prerequisite —
this matters for a uv project, since it keeps the toolchain reproducible via
`uv.lock` instead of depending on whatever is on `$PATH`.

**Config file names.** Lefthook looks for `lefthook.yml`, `lefthook.yaml`,
`.lefthook.yml`, `.lefthook.yaml`, `.config/lefthook.yml`, `.config/lefthook.yaml`,
plus TOML/JSON/JSONC equivalents. A `lefthook-local.yml` merges over the main
config for personal overrides and is conventionally gitignored.

**Structure.** Hooks contain `jobs`, `commands`, or `scripts`. Hook-level
keys: `parallel`, `piped`, `follow`, `files`, `exclude_tags`, `exclude`,
`only`, `skip`, `setup`. Per-command keys: `run`/`script`, `glob`, `files`,
`file_types`, `exclude`, `stage_fixed`, `parallel`, `piped`, `skip`, `only`,
`tags`.

**Installation is not automatic.** `lefthook install` writes the hook shims
into `.git/hooks/`. Cloning the repo does not install them — each clone must
run it. This is the main footgun: a gate nobody installed is not a gate.

### Template variables for `run`

| Variable | Expands to |
| --- | --- |
| `{staged_files}` | Files staged for commit |
| `{files}` | Result of the command in the `files` key |
| `{all_files}` | All git-tracked files matching `glob` |
| `{push_files}` | Files committed but not yet pushed (for `pre-push`) |
| `{cmd}` | The command value itself, for wrapping in a runner |

Omitting a file variable entirely runs the command with no file arguments —
which is how you make a tool check the whole project instead of a file list.

### `stage_fixed`

Automatically stages files a command modified. Intended for linters and
formatters run with a fix flag, so the fixes land in the same commit.

## Key excerpts

> Sometimes your linter fixes the changes and you usually want to commit them
> automatically.
> — https://lefthook.dev/examples/stage_fixed/

> Works only for `pre-commit` Git hook
> — https://lefthook.dev/examples/stage_fixed/

> a single dependency-free binary which can work in any environment
> — https://pypi.org/pypi/lefthook/json

Canonical `stage_fixed` example:

```yaml
pre-commit:
  commands:
    lint:
      run: yarn lint {staged_files} --fix
      stage_fixed: true
```

Glob-filtered staged-file example:

```yaml
pre-commit:
  commands:
    eslint:
      glob: "*.{js,ts,jsx,tsx}"
      run: yarn eslint {staged_files}
```

## Options considered

| Option | Pros | Cons |
| --- | --- | --- |
| Lefthook via PyPI dev dep | Version locked in `uv.lock`; no system prereq; reproducible | Extra dev dependency |
| Lefthook via system binary | Already installed here (2.1.9) | Unpinned; breaks on a machine without it |
| `pre-commit` framework | Python-native, very common, manages own tool envs | Duplicate tool versions outside `uv.lock`; slower; another config language |
| Hand-written `.git/hooks/pre-commit` | Zero dependencies | Not version-controlled; no staged-file handling; each clone re-rolls it |

## Conclusion

Use Lefthook installed as a uv dev dependency, configured in `lefthook.yml`
at the repo root. Ruff runs against `{staged_files}` with `stage_fixed: true`;
ty runs with no file variable so it sees the whole project. See the spec's
**Decisions**.

## Caveats

- **Partial staging + `stage_fixed`, tested 2026-07-25 (Lefthook 2.1.9): safe.**
  Built a file with two far-apart hunks — a staged, ruff-fixable unused import
  near the top, and a separate unstaged `UNSTAGED_MARKER` assignment near the
  bottom (staged via write-version-A-then-`git add`-then-overwrite-with-version-B,
  since `git add -p` isn't scriptable) — and ran a real `git commit` through the
  installed hook. `git show HEAD -- <file>` after the commit contained only the
  staged hunk (ruff's fix to the import); `UNSTAGED_MARKER` was absent from the
  commit and remained present, unstaged, in the working tree afterward —
  preserved, not lost or clobbered. So `stage_fixed` did not sweep unstaged
  content into the commit in this test: it re-stages only the fixer's changes
  to the file, not the whole working-tree copy. One caveat on the test itself:
  the first commit attempt failed for an unrelated reason (`ty-check` runs
  unscoped over the whole project and an unrelated untracked file in the repo
  had a real type error), which had to be sidestepped by temporarily moving
  that file out of the tree before retrying — a reminder that `ty-check`'s
  project-wide scope means any untyped file anywhere can block unrelated
  commits. This was one run on one Lefthook version; treat as evidence, not
  proof for all versions/configs.
- `stage_fixed` works only on `pre-commit`; it silently does nothing elsewhere.
- Version numbers above are current as of 2026-07-25. Recheck if Lefthook 3.x
  appears, since the config schema could shift on a major.
