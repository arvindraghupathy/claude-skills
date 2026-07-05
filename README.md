# Claude Skills

A collection of Claude Code skills for disciplined, high-quality development workflows.


---

## Skills

### `/step-by-step-debug` — Iterative Debug Mode

A structured debugging companion that refuses to guess. Before touching any code, Claude
understands the bug deeply, verifies it against evidence, and agrees on root cause with you.
Then it fixes it using the same disciplined step-by-step workflow.

1. **Interview** — questions until the bug is fully understood
2. **Screenshot** — requests visual evidence for any UI/UX issue
3. **Verify** — confirms the evidence matches the described symptoms
4. **Analyze** — identifies root cause, presents findings, waits for your agreement
5. **Fix** — full step-by-step plan → execute → subagent review → tests → summary

**Trigger:** `/step-by-step-debug`
**Exit:** `/done` or "just fix it"

---

### `/step-by-step` — Iterative Development Mode

Transforms Claude into a disciplined, methodical developer that never rushes. Instead of generating a wall of changes, it:

1. **Interviews you** — asks focused questions until both you and Claude are fully aligned on what needs to change and why
2. **Proposes a plan** — shows a full step breakdown (title + description for each) and waits for your approval before writing a single line
3. **Executes one step at a time** — makes one atomic change, explains what changed, and asks "continue?" before proceeding
4. **Proposes test scenarios** — derives a list of test cases from the actual changes made, gets your approval, then writes them one at a time
5. **Generates a change summary** — produces a clean summary of everything changed, suitable for use as a PR description

Also manages context automatically — compacts at 40% usage to keep sessions running smoothly.

**Trigger:** `/step-by-step`  
**Exit:** `/done` or "just finish it"

---

## Installation

**Personal install (all projects):**

```bash
npx skills add arvindraghupathy/claude-skills
```

Or manually:

```bash
# macOS / Linux
cp -r skills/step-by-step ~/.claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse skills\step-by-step $env:USERPROFILE\.claude\skills\
```

**Project-only install (current repo):**

```bash
mkdir -p .claude/skills
cp -r skills/step-by-step .claude/skills/
```

Then restart Claude Code. Type `/step-by-step` to activate.

---

## Requirements

- Claude Code v2.1.145 or later
- Any paid Anthropic plan (Pro, Max, Team, or Enterprise)

---

## License

MIT
