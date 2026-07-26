---
description: Execute a spec from the prompts folder via Sonnet subagents
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

5. **Implement by delegating to Sonnet subagents — one per task, in order.**
   Do not write the implementation yourself. Your job here is orchestration:
   scope each task, dispatch it, check what came back.

   For each task in the spec, in order:

   - Spawn a subagent with the `Task` tool, `model: "sonnet"` and
     `subagent_type: "general-purpose"`. Every task gets a fresh agent.
   - Run them **sequentially**, not in parallel. Later tasks usually depend on
     earlier ones, and concurrent agents editing shared files conflict.
   - **The subagent shares none of your context.** Its prompt must stand
     alone. Include: the spec path and the task's exact text, the relevant
     **Decisions** and **Research** findings (or the paths to read), the files
     it may touch, the repo conventions it must follow (`uv run` for all
     Python, never `pip` or bare `python`), what earlier tasks already changed,
     and what "done" means for this task specifically.
   - Tell it to report back what it changed, what it could not do, and
     anything it found that contradicts the spec. Instruct it not to commit,
     push, or work beyond its task.

   After each subagent returns:
   - **Check the work rather than trusting the report.** Read the files it
     claims to have changed. A subagent reporting success is evidence, not
     proof.
   - If it went wrong or overreached, correct it — either directly for a small
     fix, or by dispatching a follow-up agent with sharper scope.
   - Mark the task complete and pass forward what the next agent needs.

   If a task turns out to be wrong or blocked, stop and explain rather than
   letting an agent invent a workaround. If a subagent reports something that
   contradicts the spec, treat that as a real finding worth surfacing.

6. **Run the Verification checks yourself.** Do not delegate this — the agents
   that wrote the code are the wrong ones to grade it. Run each check for
   real and paste the actual output. If one fails, fix it (directly or via
   another subagent) and re-run. If you can't, say exactly what failed and
   why — never report a check as passing when it wasn't run.

7. Update the spec: set `**Status:**` to `done` (or `partial`), and tick the
   task and verification boxes that genuinely passed.

8. Report: what changed, what verification showed, what was left undone and
   why. Note anything a subagent got wrong that needed correcting — that's
   signal about how to scope the next one. Don't commit — the user runs `/cp`
   when ready.

9. If the user made any decision during execution — a correction, a choice
   between approaches, an override of the spec — record it in `decisions/`
   per `decisions/README.md`.

## Conventions

- Read `CLAUDE.md` and follow it.
- Check `decisions/` before asking the user anything. A logged preference is
  a default to apply, not a question to ask again.
- All implementation runs through Sonnet subagents. Verification is yours.
- Use `uv` for everything Python (`uv run`, `uv add`). Never `pip` or bare
  `python`.
- Don't commit, push, or otherwise take outward-facing actions on your own.
- Prefer the simplest thing that satisfies the spec. This is an experiments
  repo; scaffolding an experiment doesn't need is a cost, not a feature.
