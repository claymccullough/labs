---
description: Generate a numbered spec folder describing a change to make end to end
allowed-tools: Bash(ls:*), Bash(find:*), Bash(git log:*), Bash(git status:*), Bash(mkdir:*), Bash(uv pip list:*), Bash(uv tree:*), Read, Write, Edit, Glob, Grep, AskUserQuestion, WebSearch, WebFetch
---

Write a spec describing, end to end, the change requested below.

Change to spec out: $ARGUMENTS

## Context

- Existing specs: !`ls -1 prompts/ 2>/dev/null || echo "(none yet — this is 0001)"`
- Repo root: !`ls -1`

## Steps

1. If `$ARGUMENTS` is empty, ask what change to spec out and stop.

2. **New spec, or iterate the current one?** If `$ARGUMENTS` reads as feedback
   on the spec just written or named — a correction, an answer to a question
   you asked, a change of approach, an added requirement — treat it as
   iteration: revise that spec in place rather than creating a new folder.
   Only start a new spec when the request is a genuinely separate change.
   When it's unclear, ask which is meant.

   Iterating means: re-read the spec, apply the feedback, and propagate it —
   a changed approach usually invalidates tasks, file lists, and verification
   steps, not just one paragraph. Keep the same folder and index. Note what
   changed in **Decisions** and say what you revised.

   Reuse the existing **Research** rather than redoing it, but research again
   when the feedback moves onto ground it doesn't cover — a different library,
   a new API, an approach the spec hadn't considered.

3. For a new spec, pick the index: highest existing number in `prompts/` plus
   one, zero-padded to 4 digits. Add a short kebab-case slug. Create
   `prompts/NNNN-slug/`.

4. **Investigate before writing.** A spec that names the wrong files is worse
   than no spec. Read the code the change touches, follow the imports, and
   confirm every path you cite actually exists. Note what you could not
   determine rather than guessing.

5. **Research the outside world before speccing.** Don't spec from memory.
   Anything external the change depends on — libraries, algorithms, APIs,
   file formats, protocols, model names, tools — gets checked against current
   sources first. Training data goes stale: versions move, APIs break, the
   recommended approach changes, and libraries get deprecated or replaced.

   Research whatever is relevant to the change at hand:
   - **Libraries.** What's actually used for this today? Compare the real
     candidates — maintenance status, last release, API ergonomics, transitive
     weight, licence. Check whether the repo already depends on something that
     does the job before adding anything.
   - **APIs and versions.** Read the current docs for the exact version this
     repo would use, not a remembered older one. Confirm the functions and
     arguments you plan to cite exist and are not deprecated.
   - **Algorithms and approaches.** For anything non-obvious, find the standard
     approach and its complexity, failure modes, and known pitfalls before
     inventing one.
   - **Prior art.** Someone has likely solved this. Look before designing.

   Use `WebSearch` and `WebFetch` — prefer official docs, the source
   repository, and release notes over blog posts and Q&A answers. For anything
   Claude- or Anthropic-related, load the `claude-api` skill instead of
   searching from memory.

   **Check `ai_research/` first.** Past specs saved their findings there. If a
   prior note already covers what you need, read and cite it instead of
   re-researching — but confirm it's still current, and note the recheck date
   if you do. Stale notes get corrected, not silently trusted.

   **Save what you find to `ai_research/NNNN-slug/`**, matching the spec's
   folder name. One markdown file per topic, named for the subject
   (`tenacity.md`, `backoff-algorithms.md`) — not `notes.md`. Use the research
   note template below. This directory is committed: it's a durable body of
   knowledge for later specs, so write it to be useful to someone who hasn't
   read the spec.

   Record findings in your own words, plus verbatim excerpts of whatever
   actually drove a decision — API signatures, version constraints, caveats,
   deprecation notices. Enough that the finding survives the source changing
   or disappearing, without pasting whole pages.

   Then summarize in the spec's **Research** section, linking to the note
   files. If you could not verify something, mark it unverified rather than
   presenting a recollection as fact. Skip research only when the change is
   purely internal and touches nothing external — and say that's why.

6. **Surface the design decisions before writing the spec. Always.** Any spec
   worth writing contains choices that could reasonably go more than one way,
   and picking one silently buries the most important part of the plan.

   As you investigate, collect every decision where a different choice would
   produce materially different work: architecture and where things live,
   data models and formats, interface and API shape, dependencies (new library
   vs. standard library vs. hand-rolled), scope boundaries, error and edge-case
   handling, migration or backward-compatibility strategy.

   Ground these in step 5's findings — a library choice presented without
   what the research turned up is a guess wearing a recommendation's clothes.

   **Check `decisions/` first.** Past choices are logged there. If one already
   settles a question, apply it as a default and say you're doing so rather
   than asking again — re-asking a settled question wastes the log's purpose.
   Ask only when the decision is genuinely new, or when this context differs
   from the one the old decision was made in.

   Then, before writing:
   - Present each decision with the realistic options, the trade-off, and your
     recommendation with reasoning. Be concrete about consequences.
   - Use `AskUserQuestion` for the consequential ones — put your recommended
     option first, labeled `(Recommended)`.
   - If you find no such decision, say so explicitly and name the default you
     are proceeding with. Don't claim there are none just to skip the step.
   - **Log every answer** to `decisions/` per `decisions/README.md`, one file
     per decision. Record overrides of your recommendation especially — those
     carry more signal than agreements.

   Genuinely trivial or conventional choices don't need a question — decide
   them, and record them in **Decisions** so they're visible and reversible.

7. Write `prompts/NNNN-slug/spec.md` using the template below. Fill in every
   section with specifics — real paths, real function names, real commands.
   Record the resolved decisions and their rationale in **Decisions**, and
   what you verified in **Research**. Delete a section only if it genuinely
   does not apply.

8. Report the spec path and the research notes written under
   `ai_research/NNNN-slug/`, summarize the plan in a couple of sentences, and
   note any decision still open or anything left unverified. Invite feedback:
   another `/gen` with corrections will iterate this spec. Do not implement
   the change — `/exe` does that.

## Template

```markdown
# NNNN — <Title>

**Status:** draft
**Created:** <YYYY-MM-DD>
**Revised:** <YYYY-MM-DD — what changed; omit until first revision>

## Goal

What should be true when this is done, and why it's worth doing. State the
outcome, not the implementation. If there's a motivating problem, name it.

## Context

What someone needs to know before touching this — how the relevant code
works today, constraints, prior attempts. Link to files with `path:line`.

## Research

What was checked externally, and what it means for this spec. Full notes in
`ai_research/NNNN-slug/`.

- **<Library/API/approach>** — the finding in one or two lines, and why it
  matters here. → [`ai_research/NNNN-slug/<topic>.md`](../../ai_research/NNNN-slug/<topic>.md)
- **Reused:** <finding> — from an earlier spec's note, rechecked YYYY-MM-DD.
- **Unverified:** <what, and why it couldn't be confirmed>

## Decisions

Every architectural or design choice this spec makes, and why. One entry per
decision: what was chosen, what was rejected, and the reasoning — referencing
**Research** where the choice rests on it. Mark whether it was confirmed by
the user or defaulted, so a defaulted choice can be challenged later.

- **<Decision>:** chose X over Y because Z. *(confirmed | defaulted)*

## Files

### Read
- `path/to/file.py` — why it matters

### Write
- `path/to/file.py` — what changes, and roughly how
- `path/to/new_file.py` — new; what it holds

## Tasks

Ordered, each independently completable. Enough detail to act on without
re-deriving the design; not so much that it's the diff written twice.

1. [ ] ...
2. [ ] ...

## Verification

How to know it actually worked — concrete commands and their expected
results, plus anything needing a human eye.

- [ ] `uv run pytest tests/test_x.py` — passes
- [ ] `uv run ruff check .` — clean
- [ ] ...

## Out of scope

Things deliberately not being done, so they don't creep in.

## Open questions

Anything unresolved that could change the approach. Empty is fine.
```

## Research note template

One file per topic in `ai_research/NNNN-slug/`. Write for someone who hasn't
read the spec and may land here from a later one.

```markdown
# <Topic>

**Researched:** <YYYY-MM-DD> · **For:** [NNNN-slug](../../prompts/NNNN-slug/spec.md)
**Sources:** <url> · <url>

## Question

What this was trying to settle.

## Findings

What's true, as of the date above. Versions, API shapes, constraints,
gotchas. Be specific — "requires Python >=3.9" beats "needs a recent Python".

## Key excerpts

Verbatim from the source, for the parts a decision rests on. Quote sparingly
and attribute each one.

> <quote>
> — <source url>

## Options considered

Where there were alternatives: what they were, and how they compare.

| Option | Pros | Cons |
| --- | --- | --- |

## Conclusion

What this implies for the work, and what was chosen. Cross-reference the
spec's **Decisions** entry.

## Caveats

What's unverified, version-sensitive, or likely to go stale. Say what would
need rechecking and roughly when.
```
