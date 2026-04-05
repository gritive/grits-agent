---
description: "Complete current work: verify → commit → update Grits → suggest next. Triggers on: 'done', 'task complete', 'finish work', 'grits done'."
---

# Grits Complete Work Workflow

## Step 1: Verify Changes

If there are code changes, run the project's verification commands (lint, build, test).
Check the project's README or CI config for the exact commands.

Fix and retry on failure. Never mark as complete without passing verification.

## Step 2: Commit (On User Request)

If the user wants to commit, commit the changes. Reflect the work in the commit message.
**Only push when the user explicitly requests it.**

## Step 3: Update Grits (Required)

Mark the in-progress Grits task as complete. **Always use `work_done`** (not `task_done`) — it updates the description with results before closing.

```
work_done(
  task_id: "<TASK_ID>",
  message: "One-line summary of what was done",
  description: "<existing description>\n\n## Results\n- Changes: ...\n- Files modified: ...\n- Verification: ..."
)
```

**Description rules:**
1. Preserve existing sections (## Problem, ## Approach, etc.) as-is
2. Append a `## Results` section at the end
3. Include: changes made, files modified, verification outcome (tests pass/fail)

**Good message examples:**
- "Fixed redirect URL mismatch between email link and actual route path"
- "Batched N+1 dashboard queries: 100+ → 3 queries for 20 projects"
- "Added smart_view filter to task context. Excludes other users' IN_PROGRESS tasks"

**Bad message examples:**
- "Fixed"
- "done"
- "Bug fix"

## Step 4: Suggest Next Task

```
task_next()
```

Show the next recommended task and ask if the user wants to continue.

## Important Notes
- Never mark as complete without passing verification (test/lint)
- Commit only when the user explicitly requests it
- Push only when the user explicitly requests it
- **Never skip the Grits update** — always call task_done after code changes
- Completion comments must include **what, why, and how** — others should be able to read it later
