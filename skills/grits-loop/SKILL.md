---
name: grits-loop
description: Use when you want Claude Code to autonomously advance a Grits project's tasks through their kanban states (todo → in_progress → in_review → done) on a recurring basis. Triggered by `/grits-loop <project>`, typically wrapped in `/loop` for cadence. One invocation = one tick over that project.
---

# grits-loop

## Overview

Runs **one tick** of an autonomous development loop over a single Grits project. Each task's kanban state is the control plane; Grits is the durable state store. The main tick is a **thin dispatcher**: it reads Grits state and delegates **every unit of real work — analysis, build, AND the `done`-gate land+deploy — to a subagent**, reading back only short summaries. It never runs build/test/deploy/git/`/ship`/`/land-and-deploy` in its own context. The human is in the loop **asynchronously** via task state + Remote Control messages.

```
/grits-loop                    # one tick on the current directory's project (.grits.json)
/grits-loop grainfs            # one tick on an explicit project slug
/loop 30m /grits-loop          # recurring (every 30 min), current directory's project
```

The argument is the target Grits project **slug** (e.g. `grainfs`). **It is optional**: with no argument, the loop targets the project configured for the current directory — the same project `grits task list` uses (read from `.grits.json`'s `project_id`).

## Hard rule (non-negotiable)

**NEVER call `AskUserQuestion` or any blocking prompt.** A blocking prompt cannot be answered remotely (it blocks stdin) and stalls the loop with no remote escape. When a decision is needed, route it through Grits async (see *Blocked handling*). Violating this defeats the entire design.

## Prerequisites (verify once, warn if missing)

- `settings.json`: `{"permissions":{"defaultMode":"auto","deny":["AskUserQuestion"]}}` — eliminates blocking permission prompts and forces plain-text questions.
- Remote Control active (`claude --remote-control` or `/remote-control`) + `agentPushNotifEnabled: true` — so questions reach your phone and you can reply remotely.
- `gh`/`git` authenticated; `/land-and-deploy` configured for the project (its deploy step, e.g. via `/setup-deploy`) — for the in_review→done land/deploy step. The skill is project-agnostic; the concrete deploy command lives in each project's own config, **never hardcoded here**.

If a prerequisite is missing, state it plainly and continue with what works (e.g. no Remote Control = answer at the laptop).

## One tick

1. **Resolve project**:
   - **Argument given** → treat it as a slug, resolve slug → id via `mcp__grits__project_list`. If not found, report and stop.
   - **No argument** → read `project_id` from the current directory's `.grits.json` (`cat .grits.json`; this is the project `grits task list` targets). Confirm it against `mcp__grits__project_list` to get the name for logging. If `.grits.json` is absent or has no `project_id`, report plainly ("no project argument and no `.grits.json` in this directory — run `grits init` or pass a slug") and stop. **Do not** call `AskUserQuestion`.
   - Re-resolve every tick (no cross-tick caching).
2. **Read tasks**: `mcp__grits__task_list` scoped to that project.
3. **Collect open decisions**: note every existing `needs-info` task — its question is still open. Do **not** re-process; they are consolidated in step 5.
4. **Dispatch handlers by state** for every actionable task (see *Concurrency*). **The main tick does not do the work itself — for each actionable task it dispatches a subagent and reads only the returned summary** (this includes the `done`-gate land+deploy). A task that blocks does **not** stop the tick — record it (see *Blocked handling*) and keep going.
5. **End of tick — consolidate ALL open decisions** (existing `needs-info` from step 3 **plus** tasks newly blocked this tick) into one batch: `❓ DECISIONS NEEDED:` then one line per task, framed explicitly as needing a decision (raises the odds the push fires). Then end the turn. If there are no open decisions, end silently.

## State → handler (idempotency markers in **bold**)

The states below are Grits **`status_category`** values — read them with `task_list`/`task_get`, set them with `task_update status=<CATEGORY>`: `todo`=`TO_DO`, `in_progress`=`IN_PROGRESS`, `in_review`=`IN_REVIEW` (the workspace's "리뷰" / "In Review" column), `done`=`DONE`. Moving a task to `in_review` is a **real status transition**, not a comment label. (`needs-info` is not a status_category — track it per *Blocked handling*.)

| State | Action | Skip when (already done) |
|---|---|---|
| `todo` | subagent runs the **analysis half** of the task's engine skill (see *Skill routing by tag*) → write spec + plan back to Grits (`task_update` description/comment), notify human | **spec already present** |
| `in_progress` (no PR) | subagent runs the **build half** of the engine skill → open PR via `/ship`; then main sets status to **`IN_REVIEW`** (`task_update status=IN_REVIEW`) and records the PR link via `task_comment` | **already `in_review` / PR link present** → instead: address new review comments, push to the existing PR, set status back to `IN_REVIEW` |
| `in_review` | **none** — human reviews the PR | always (human's turn) |
| `done` (not merged) | **dispatch a subagent** that runs the project's land+deploy (merge to base → project's pre-push gate → push → project's deploy command → health check) and returns a **short verdict** → main records merged via `task_comment` | **already merged** |
| `needs-info` | collected in step 3 for the end-of-tick batch — no other action | always |

"notify human" and "record merged" mean a `task_comment` (never a blocking prompt).

- **todo gate**: after spec is written, the task stays in `todo` awaiting human review. The human editing/commenting then dragging `todo → in_progress` **is** the spec+plan approval. Never auto-advance todo→in_progress.
- **done gate**: only the human moves a task to `done`. That deliberate act **is** the authorization to land + deploy. The handler then **dispatches a subagent** that runs the project's land+deploy non-interactively — using the project's own commands (`/land-and-deploy` if the project uses PRs, or its documented deploy such as `make deploy` for direct-to-base repos — **never hardcoded here**, read from the project's config). The subagent keeps the heavy logs (test output, deploy output) in its own context and returns only a short verdict (merge SHA, deploy status, health result). If a pre-push gate fails or a decision is needed, the subagent returns `BLOCKED:` / a failure summary and main routes the task to `needs-info` — it does **not** push through. This mirrors the manual flow: `/ship` opens the PR (→ `in_review`), the human moving `in_review → done` triggers the land/deploy.

## Skill routing by tag

Each task is driven by **one engine skill**, split across the `todo → in_progress` human gate (todo = analysis half, in_progress = build half):

| Tag | Engine | `todo` half (analysis) | `in_progress` half (build) |
|---|---|---|---|
| `bug` / `버그` | **investigate** | root-cause diagnosis + reproducing test + fix plan → Grits | implement the fix → `/ship` → `in_review` |
| anything else | **feature-pipeline** | spec + plan (its early phases) → Grits | implement → code review → `/ship` (its Phase 8) → `in_review` |

- **`/ship` is the single PR-open step for both paths.** feature-pipeline ends at ship by design; investigate does **not** ship itself, so the bug `in_progress` handler invokes `/ship` explicitly after the fix lands. Result: both paths reach `in_review` with a PR.
- **Never merge or run `/land-and-deploy` in the build half.** Both engines stop at the open PR. Land + deploy (`/land-and-deploy`, which merges + deploys per the project's config) happens **only** at the `done` gate, after the human moves `in_review → done`.
- **Run the engines non-interactively — suppress their built-in gates.** feature-pipeline's spec/plan/code-review gates and investigate's AskUserQuestion are **replaced** by grits-loop's gates: the human `todo → in_progress` drag IS the spec/plan approval, and any mid-build decision routes to `needs-info` (see *Blocked handling*), never a blocking prompt. Do not let an engine's interactive phase (office-hours/brainstorming/grill-me/review-forever) emit a prompt inside the loop.

## Blocked handling (replaces AskUserQuestion)

When a subagent cannot proceed without a human decision, it returns `BLOCKED: <question> / options: <A|B>` as its final result. The main agent then:

1. `task_update` + `task_comment`: move the task to `needs-info`, record the question.
2. **Accumulate** the question and **continue processing the other tasks**. Do NOT end the turn here — one blocked task must not starve the rest of the tick.

All accumulated questions are emitted together at *End of tick* (step 5), then the turn ends. The push notification reaches the phone; the human replies via Remote Control message whenever. Their reply is handled out-of-band (next section), not inside a tick.

## Handling human replies (out-of-band)

A human reply ("task X → option B", or PR review feedback) arrives as a normal message between ticks. When you receive one:

1. Apply it: `task_update` to clear `needs-info`, record the decision.
2. Re-dispatch the relevant handler subagent for that task (resume with the answer in its prompt; or `SendMessage` the original subagent to keep context).

State lives in Grits, so a `/loop` re-fire never loses a pending decision — it just sees `needs-info` and waits.

## Concurrency

Dispatch independent tasks' subagents in parallel. **Serialize implementation** (`in_progress`) tasks, or give each its own git worktree, to avoid merge conflicts. For MVP, one `in_progress` implementation at a time is safest.

## Subagent rules

- Subagents do the heavy work in isolated context; the main agent stays thin (resolve → dispatch → read short summaries → relay). This keeps main context small.
- **Every state's real work runs in a subagent — including the `done`-gate land+deploy.** If you (the main tick) find yourself running `git merge`/`git push`/`make`/test/build/deploy/`/ship`/`/land-and-deploy` directly, stop: that work belongs in a subagent that returns a short verdict. The main tick only reads Grits, dispatches, and relays.
- The engine skill is chosen by tag (*Skill routing by tag*): `bug`/`버그` → `investigate`; everything else → `feature-pipeline`. The bug `in_progress` subagent must invoke `/ship` after the fix — `investigate` does not open a PR on its own.
- Run engines **non-interactively**. **Never** let an interactive phase emit a prompt (`brainstorming`, `office-hours`, `grill-me`, `plan-eng-review`, `review-forever` gates, or `AskUserQuestion`) — route every decision through `needs-info` (Blocked handling) instead.

## Common mistakes

- **Calling AskUserQuestion** → loop stalls, unanswerable remotely. Use Blocked handling.
- **Ending the tick the instant one task blocks** → starves all other actionable tasks. Record the block, keep processing, batch all questions at end of tick.
- **Re-speccing a todo that already has a spec** / **opening a second PR for an in_progress task with a PR** → missing idempotency check. Always check the skip marker first.
- **Auto-advancing todo→in_progress or auto-merging before done** → removes the human gates. Only the human makes those two transitions.
- **Acting on in_review tasks** → that's the human's review turn; skip them.
- **Bug fix that never opens a PR** → `investigate` fixes but does not `/ship`. The bug `in_progress` handler must run `/ship` itself, or the task is stuck (fixed locally, never reaches `in_review`).
- **Leaving a shipped task in `in_progress`, or faking `in_review` with only a comment** → `in_review` is a real Grits status (`IN_REVIEW`, the "리뷰" column). After `/ship`, transition it with `task_update status=IN_REVIEW`. Don't rationalize "this workspace has no review status" — verify with `task_list`/`task_get` before inventing a marker.
- **Letting feature-pipeline merge/deploy** → it stops at the PR; `/land-and-deploy` and merge are the human's `done` gate, never the in_progress step.
- **Running the tick's real work in the main context** (merge, push, tests, build, `/ship`, deploy, `/land-and-deploy`) → main context balloons (test/deploy logs pile up) and the design breaks. The main tick reads Grits + dispatches a subagent per task + relays summaries; the subagent does the work and returns a short verdict — this includes the `done`-gate land+deploy.
