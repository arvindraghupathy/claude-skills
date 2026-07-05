# Execution & Review Mechanics

Detailed mechanics for Phase 3 (Execute). Load this when executing steps.

## Step Announcement

```
**Step N of M:** [Title]
[1–2 sentences: what exactly is changing and why this is the right move at this point]

Making the change now...
```

## After Execution — Report and Offer Review

Report what changed, then offer the user a choice on whether to run a review:

```
✅ Done. [1–2 sentences: what changed and where]

🔍 Run a code review on this step?
  Y) Yes — check for repo violations, correctness, DRY, and modularity
  N) No — continue to next step
```

To avoid nagging, ask the user once how they want reviews handled and remember it for the
session: "review every step", "never review", or "ask me each time". Only fall back to the
per-step Y/N prompt if they chose "ask me each time" or haven't expressed a preference.

Wait for the user's response before doing anything else.

- **N / "continue" / "yes" / "go"** → proceed directly to the next step.
- **Y** → spawn the reviewer subagent (below).

## Reviewer Subagent

Spawn a reviewer subagent using the `Task` tool. It runs in its own clean context so it has
no bias toward the implementation decisions made in this session.

**Reviewer prompt (pass verbatim, substituting bracketed values):**

```
You are a code reviewer. Review the changes made in this step and report findings only —
do not fix anything yourself.

Step just completed: [step title and one-sentence description]
Files changed: [list of files modified or created]

Review the current state of those files and the broader codebase and assess:

1. REPO VIOLATIONS — Does anything break established patterns, conventions, naming rules,
   or architectural decisions visible elsewhere in the repo? Cite specific examples.
2. CODE CORRECTNESS — Are there bugs, logic errors, unhandled edge cases, missing null
   checks, or incorrect assumptions? Be specific.
3. DRY VIOLATIONS — Is any logic duplicated that already exists elsewhere in the repo?
   Look broadly, not just in the changed files.
4. MODULARITY — Are responsibilities well-separated? Is anything doing too much? Are there
   obvious seams where future enhancements would require touching too many places?

Format your response as:
REPO VIOLATIONS: [findings, or "None"]
CORRECTNESS: [findings, or "None"]
DRY: [findings, or "None"]
MODULARITY: [findings, or "None"]
VERDICT: PASS | NEEDS FIXES

Be concise. One line per finding. If a category is clean, say "None".
```

**Handle the verdict:**

If PASS:
```
🔍 Reviewer: Pass — no issues found.
Shall I continue to Step [N+1]?
```

If NEEDS FIXES:
```
🔍 Reviewer flagged issues:
- REPO VIOLATIONS: [findings if any]
- CORRECTNESS: [findings if any]
- DRY: [findings if any]
- MODULARITY: [findings if any]

How would you like to proceed?
  A) Fix the issues before moving on (recommended)
  B) Log them and continue — I'll address at the end
  C) Dismiss — these are acceptable trade-offs
```

Wait for the user's choice. If A, fix the issues then re-run the reviewer on the fixed
version before marking the step complete. If B, add to the Review Log for the final summary.
If C, note the dismissal and move on. Then stop — do not proceed until the user responds.

## Review Log

Maintain a running list of issues the user chose to defer (option B). Include it in the
Phase 5 summary under **Deferred Review Items**.

## Handling a Broken Build or Failing Change

If a change breaks the build, fails to compile, or throws at runtime:
- Stop immediately. Do not proceed to the next step.
- Report the error plainly and attempt a fix within the current step's scope.
- If the fix requires expanding scope beyond the current step, flag it and get user approval.
- Never paper over an error by silently changing unrelated code.

## Handling Mid-Execution Feedback

- **"Continue" / "yes" / "go"** → announce next step, execute, report, wait
- **"Run X and Y in parallel"** → execute both in one response, label each change clearly, summarise both, then ask to continue
- **"Why did you do it that way?"** → explain; don't proceed until they're satisfied
- **"Change X first"** → incorporate feedback, adjust remaining plan if needed, confirm change, continue
- **"Skip this step"** → acknowledge, note any implications, move to next step
- **"Revise the plan"** → pause execution, show updated remaining steps, get re-approval
- **"Just finish it"** → exit step-by-step mode, complete remaining steps, summarize all changes

## Parallelism

Two steps are parallelisable only if: they touch different files with no shared dependencies,
neither's output feeds the other, and running them together cannot cause a conflict or
inconsistent state. In a single chat session "parallel" means executing them in one response
to save round-trips — not true concurrency. If parallel groups exist, offer them after the
plan; if none, say nothing.
