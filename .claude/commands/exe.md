---
description: Execute a spec from the prompts folder
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Task, TaskCreate, TaskUpdate, WebSearch, WebFetch, AskUserQuestion
---

Execute the spec identified below, following `CLAUDE.md`.

Spec to execute (number, slug, or empty for the most recent): $ARGUMENTS

## Context

- Available specs: !`ls -1 prompts/ 2>/dev/null || echo "(none — run /gen first)"`
- Working tree: !`git status --short`

## Steps

1. Resolve the spec. `$ARGUMENTS` may be a number (`3`, `0003`), a slug, or
   empty — if empty, use the highest-numbered spec and say which you picked.
   If it's ambiguous or missing, list the options and stop.

2. Read `prompts/NNNN-slug/spec.md` in full, then read every file in its
   **Read** list before changing anything. The spec is a plan, not ground
   truth: if it contradicts the code, trust the code and say so.

   Read **Decisions** carefully — they're the reasoning behind the plan, not
   trivia. If a decision marked *defaulted* looks wrong now that you've read
   the code, or an **Open question** would change the approach, raise it
   before implementing rather than quietly picking. Don't relitigate choices
   marked *confirmed* unless the code shows they can't work.

   Read the notes in `ai_research/NNNN-slug/` too — they hold the API shapes,
   version constraints, and caveats the spec was built on, and will save you
   re-deriving them. Check their **Caveats**: if a note is old enough that its
   findings may have moved, verify before relying on it. When implementation
   turns up something the notes got wrong or missed, correct the note as part
   of the work.

3. If the working tree has unrelated uncommitted changes, point them out
   before adding more.

4. Track the tasks. Use TaskCreate/TaskUpdate for multi-step specs so
   progress is visible. Mark tasks complete as you finish them.

5. Implement, one task at a time.
   - Match the conventions of the surrounding code.
   - Build only what the spec asks for. If you find adjacent problems, note
     them for later instead of fixing them mid-stream.
   - If a task turns out to be wrong or blocked, stop and explain rather
     than inventing a workaround.

6. Run the **Verification** checks. Run them for real and paste the actual
   output. If one fails, fix it and re-run. If you can't, say exactly what
   failed and why — never report a check as passing when it wasn't run.

7. Update the spec: set `**Status:**` to `done` (or `partial`), and tick the
   task and verification boxes that genuinely passed.

8. Report: what changed, what verification showed, what was left undone and
   why. Don't commit — the user runs `/cp` when ready.

## Conventions

- Read `CLAUDE.md` and follow it.
- Use `uv` for everything Python (`uv run`, `uv add`). Never `pip` or bare
  `python`.
- Don't commit, push, or otherwise take outward-facing actions on your own.
- Prefer the simplest thing that satisfies the spec. This is an experiments
  repo; scaffolding an experiment doesn't need is a cost, not a feature.
