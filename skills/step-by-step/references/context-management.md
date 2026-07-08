# Context Management

Load this for the details of keeping long sessions sharp.

## Important Limitation

Claude cannot read its own context usage as a number. Any "monitor and compact at X%"
instruction is unreliable — Claude cannot reliably detect when it is approaching a threshold.
The strategies below work around this honestly: keep responses lean, checkpoint state, and
prompt the user to compact proactively.

## Keep Responses Lean

- Step summaries: 1–2 sentences max. Not a transcript of every decision.
- Do not re-quote file contents in responses. Reference by filename only.
- Do not repeat the approved plan in full after every step. Reference step number only.
- After a step completes, retain only: what changed, which file, one-line reason.

## Session State Block

Emit a SESSION STATE block only at compaction-adjacent moments (right before prompting the
user to `/compact`), not after every single step — each block supersedes the previous one,
so emitting one per turn just accumulates noise.

```
<!-- SESSION STATE
problem: [one sentence]
plan: [M steps total, steps X–Y remaining]
remaining: [step titles only, comma separated]
decisions: [scope changes or skipped steps, one line each]
modified: [file paths, comma separated]
-->
```

## Prompting the User to Compact

Claude cannot run `/compact` itself — only the user can. At natural pause points, prompt with
a short notice:

```
💬 Good time to run `/compact` to keep the session sharp before we continue.
```

Prompt at these moments:
- After the shared understanding summary is confirmed (end of Phase 1)
- After the plan is approved (end of Phase 2)
- At each execution checkpoint (per batch — every step in Fine, every 3 in Balanced, every 5 in Fast)
- After every 3rd completed test

Keep it brief. If the user compacted recently, skip that checkpoint. Community guidance is to
compact proactively (around 60% capacity) rather than waiting for the near-full warning, since
quality degrades before the warning appears.

## The /compact Command (user-run)

| Command | What it does |
|---|---|
| `/compact` | Summarizes conversation history, freeing context. Claude re-reads the SESSION STATE block to resume. |
| `/compact [instructions]` | Same, but steers what to preserve, e.g. `/compact keep the approved plan and modified file paths` |

## After Compaction

Re-read the most recent SESSION STATE block to restore the problem, the plan and remaining
steps, user decisions, and modified files. Confirm briefly: "Compacted. Resuming at Step N —
[title]. Ready to continue?" Do not re-read large files unless a step needs it — re-fetch on
demand only.
