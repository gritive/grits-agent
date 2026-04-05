---
description: "Check next tasks → select → write description → start. Triggers on: 'what should I work on?', 'start work', 'grits start', 'next task'."
---

# Grits Start Work Workflow

## Step 1: Gather Current Status

**Collect tasks from 4 sources.** Use MCP tools for all queries.

```
# 1. Current context (my in-progress + today) — tasks to continue
task_context()

# 2. TO_DO (already planned)
task_list(status: "TO_DO", page_size: "10")

# 3. BACKLOG (not yet planned)
task_list(status: "BACKLOG", page_size: "10")
```

Also check for code TODOs/FIXMEs:
```bash
grep -rn "TODO\|FIXME" src/ --include="*.ts" --include="*.js" --include="*.py" --include="*.go" | head -10
```

Present to the user:
1. **In Progress** — tasks that can be continued (check current implementation state)
2. **Today/Scheduled** — tasks with deadlines
3. **To Do (TO_DO)** — already planned tasks
4. **Backlog** — not yet planned tasks
5. **Code TODOs** — TODO/FIXME comments in code
6. **New Task** — something that just came up

Let the user select by number.

**If there are in-progress tasks**: Check the code to assess current implementation state and suggest "Continue this?"

**Feasibility assessment**: For each task, determine "Can this be implemented/fixed in this session?"
- Solvable via code changes → Feasible (bug fixes, UI improvements, feature additions, etc.)
- Requires external dependencies (third-party integrations, infra changes) → Planning only
- Needs design/spec discussion → Decide after discussing with user
Prioritize feasible tasks. Don't skip "Feature" tasks — suggest them if they're implementable.

**Note**: Exclude tasks that are IN_PROGRESS by someone else (`task_context` automatically shows only your tasks).

## Step 2: Write Task Description (Required)

Before starting work, **write a description for the task.** Make it understandable for other team members:

- **Problem/Background**: Why is this task needed?
- **Approach**: How will it be resolved?
- **Scope**: Which files/modules are affected?

If the existing task already has a description, just supplement it. For new tasks, always write one.

## Step 3: Start Work

Start work using MCP tools:

```
# Existing task
work_start(task_id: "<TASK_ID>", description: "## Problem\n...\n## Approach\n...\n## Scope\n...")

# New task
work_start(title: "Title", description: "## Problem\n...\n## Approach\n...\n## Scope\n...")
```

**Check the linkage section in the work_start response:**
- If `has_milestone` or `has_kr` is false, review the suggestions
- If an item has `recommended: true`, link it:
  - `milestone_link(task_id: "<id>", milestone_id: "<id>")`
  - `kr_link(task_id: "<id>", kr_id: "<id>")`

## Step 4: Create New Task (If Selected)

Look up tags dynamically:

```
tag_list()
```

**Tag selection guide** — always assign a tag matching the nature of the work:
- **Bug fix** (something broken or not working) → Bug tag
- **New feature** (adding something that didn't exist) → Feature tag
- **Improvement** (making something existing better) → Improvement tag

## Step 5: Implementation (Leverage Subagents)

Implement the selected task by **actively using subagents**:

### Analysis Phase
- Delegate code/file exploration to **Explore subagents** (prevents main context pollution)
- If multiple tasks were selected, run code analysis for each via **parallel subagents**

### Implementation Phase
- Independent tasks → **worktree isolation** subagents for parallel implementation
- Dependent tasks → sequential processing
- Give each subagent exactly one clear objective

### Verification Phase
- After implementation, run integration verification in the main context

### Subagent Prompt Guide
```
[Task title]
- Objective: (from task description)
- Target files: (from scope)
- Constraints: Follow project conventions, include tests
- Deliverable: List of modified files and change summary
```

## Important Notes
- **Never skip writing a description** — others must understand the why and how
- `--assignee me` is handled automatically by `work_start`
- If multiple tasks are in progress, suggest focusing on one
