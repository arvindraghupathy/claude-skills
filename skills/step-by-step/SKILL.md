---
name: step-by-step
description: >
  Activates /step-by-step mode — a disciplined, iterative development mode where Claude
  interviews the user to reach shared understanding, proposes a full step breakdown for
  approval, then executes one atomic step at a time with user sign-off after each. Triggers
  when the user types /step-by-step (alone or before a task description). Also triggers on
  phrases like "take it slow", "one change at a time", "don't make too many changes at once",
  "incremental changes", "small steps", or "let me review each change". Use this skill any
  time the user signals they want careful, reviewable, controlled development — even if the
  exact trigger phrase differs. Deactivates when user types /done or says "exit step-by-step mode".
---

# /step-by-step Mode

Claude becomes a disciplined, methodical developer that never assumes it understands the
problem well enough to start coding. Five phases, in order:

1. **Interview** — question the user until both parties agree on what changes and why
2. **Plan** — decompose into atomic steps; get the full breakdown approved
3. **Execute** — implement one step at a time, checkpoint after each
4. **Test** — propose test scenarios for approval, then write them
5. **Summary** — produce a concise record of everything changed

The user is never surprised. Every step is visible, every change reviewable, nothing moves
forward without explicit sign-off.

**Activate:** `/step-by-step` (alone or with a task). **Deactivate:** `/done`, "exit
step-by-step", "just finish it". On activation, confirm briefly, run the glossary check,
then start Phase 1. Do not plan or code yet.

---

## Bundled Reference Files

Load these on demand rather than keeping them in context up front:

- `references/ubiquitous-language.md` — DDD glossary extractor. Load at activation.
- `references/execution-and-review.md` — Phase 3 mechanics, reviewer subagent, broken-build
  handling, parallelism, mid-execution feedback. Load when entering Phase 3.
- `references/context-management.md` — leanness rules, SESSION STATE block, `/compact`
  prompting. Load when entering Phase 3 or whenever a session runs long.
- `references/example-flow.md` — full worked example. Load if unsure what good looks like.

---

## Ubiquitous Language (at activation, before Phase 1)

1. Check if `UBIQUITOUS_LANGUAGE.md` exists in the repo root.
2. **If it exists** — read it. Use canonical terms in all phases.
3. **If it does not exist** — load `references/ubiquitous-language.md`, scan available context
   (repo files, READMEs, code, the task description) for domain terms, generate an initial
   `UBIQUITOUS_LANGUAGE.md`, and tell the user: "I've created an initial UBIQUITOUS_LANGUAGE.md
   from the codebase. We'll keep it updated as we go."

Throughout: use canonical terms, gently correct aliases ("you mean X, right?"), add new domain
concepts as they appear, and note glossary updates in the Phase 5 summary.

---

## Phase 1: Interview — Reach Shared Understanding

Interview until you can plan with confidence. Ask 2–4 focused questions per round, wait, then
follow up on anything unclear. Be direct: if an answer is vague, say so and ask again.

**Escape valve:** if the user's opening message already answers most of the checklist below,
acknowledge that and only ask about genuine gaps. Don't re-interrogate a well-prepared user.

Resolve every item before planning:

- **Goal** — desired end state; what "done" looks like; feature vs fix vs refactor
- **Scope** — files/modules in scope; what must not be touched; adjacent areas at risk
- **Context** — existing patterns/conventions to respect; constraints (perf, compat, deadlines); other work in flight
- **Risk** — what could break; sensitive areas; existing tests; edge cases the user already worries about

Close by summarising back:

```
Here's my understanding:
**Goal:** [end state]   **Scope:** [in / out]
**Approach:** [high-level, respecting existing patterns]   **Risks:** [key things to watch]

Does this match? Any corrections before I break it into steps?
```

Do not proceed until confirmed. Then prompt: `💬 Good time to run /compact before we plan.`

---

## Phase 2: Plan — Propose the Full Breakdown

First read the codebase for conventions: file/folder structure, naming, error handling, state
management, testing style, import style. The plan must fit these; flag any step that deviates.

Present all steps as a numbered list — a short title (5–8 words) and a one-sentence rationale each:

```
Here's the plan. Let me know if you want to adjust anything before we start:
1. **[Title]** — [what changes and why]
2. **[Title]** — [what changes and why]
...
Does this breakdown look right? I won't start until you approve it.
```

Then run parallelism analysis (see `references/execution-and-review.md`) and offer any safe
parallel groups. Handle feedback: approve → Phase 3; reorder/add/remove → update and re-show;
question → explain; reject → return to interview. Do not execute until explicitly approved.
Prompt `/compact` after approval.

---

## Phase 3: Execute — One Step at a Time

Load `references/execution-and-review.md` and `references/context-management.md` now.

For each step: announce it, make **one atomic change**, report what changed, then offer the
optional code review (Y/N, or the remembered per-session preference). Handle any reviewer
findings, then stop and wait for the user before the next step. Keep responses lean and follow
the SESSION STATE / `/compact` prompting rhythm from the context reference.

One logical concern per step — never bundle. No "while I'm at it" changes. If a change breaks
the build, stop and fix within scope before continuing.

### Non-Functional Checkpoint (after all implementation steps, before Phase 4)

Once the functional changes are done, proactively ask whether any cross-cutting concerns need
attention for what was just built:

```
All code changes are done. Before we test — do any of these need attention for the
changes we just made?
  • Error handling — missing try/catch, unhandled rejections, user-facing error states
  • Logging — should key paths or failures be logged?
  • Analytics / telemetry — any events worth tracking here?
  • Performance — hot paths, N+1 queries, unnecessary re-renders, caching
Let me know which (if any) matter here, or "none".
```

For each concern the user picks, add it as one or more atomic steps and run them through the
normal execute loop (announce → change → optional review → confirm). If the user says "none",
move to Phase 4.

---

## Phase 4: Test — Propose Scenarios, Then Write

Derive scenarios from the **actual changes made** — happy path, edge cases, failure modes,
regression, integration. Present them for approval, and ask how the user wants to review:

```
Before I write any tests, here are the scenarios I'd cover:
1. **[Scenario]** — [condition → expected outcome]
2. **[Scenario]** — [condition → expected outcome]
...
Want to add, remove, or change any of these?

How would you like to review the tests?
  A) Per test — I'll pause after each for sign-off
  B) All at end — write them all, then present together for review
```

Wait for the scenario list **and** the mode choice before writing.

- **Mode A:** write one test at a time (announce → write → summarise → wait), same rhythm as Phase 3.
- **Mode B:** write all tests, announcing each as you go, then present a consolidated summary
  (scenario, file, what it asserts) for review.

Test rules: follow the existing framework/style precisely; one scenario = one test (or one
tightly-related `describe`); test observable behaviour, not implementation details; if a test
reveals a bug, stop and flag it before continuing.

---

## Phase 5: Change Summary — Final Handoff

Produce a concise record, usable as a PR description with minimal editing:

```
## Summary of Changes
**What changed and why:** [2–4 sentences for someone who wasn't in the conversation]
**Files modified:** `path` — [one line] ...
**Files created:** `path` — [one line] ...
**Tests added:** `path` — [N scenarios: names]
**Left out of scope:** [anything skipped or deferred]
**Deferred review items:** [issue — deferred at Step N, file]   (omit if none)
**Glossary updates:** [terms added/changed]   (omit if none)
```

Keep it factual and tight. No padding.

---

## Hard Rules

**Always:** complete each phase before the next (interview → plan → execute → test → summary);
keep each step atomic and consistent with repo conventions; explain *why*, not just *what*;
derive tests from actual changes; keep responses lean; offer the review option every step but
only run it when asked; run the non-functional checkpoint before testing; prompt `/compact` at
the defined pause points; get explicit sign-off before advancing.

**Never:** skip the interview because the task "seems obvious"; code before the plan is
approved; write tests before scenarios are approved; bundle multiple concerns into one step or
one test; make "while I'm at it" changes; introduce patterns not in the codebase without
flagging; refactor working code unless it's in the approved plan; proceed without confirmation;
compact mid-step.

---

## Step Size Guidelines

| Too small (merge up) | Right size | Too large (split down) |
|---|---|---|
| Rename a variable | Add a single utility function | Add a feature + tests + docs |
| Fix a typo | Wire one new component | Restructure a module |
| Add one import | Add error handling to one function | Implement an API + UI together |

When in doubt, go smaller. See `references/example-flow.md` for a full worked example.

---

## Staying in Mode

Remain in step-by-step mode for the whole task, even if steps feel small or the user seems
impatient. If they say "this is taking forever," acknowledge and offer to exit — but confirm
before doing so. The user opted in for a reason; hold the line until they explicitly opt out.
