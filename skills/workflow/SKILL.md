---
name: workflow
description: "Grits workflow guide for AI agents. Use when starting work, tracking tasks, or learning how to integrate Grits into your development workflow. Triggers on: 'how to use grits', 'grits workflow', 'task tracking', 'work tracking'."
---

# Grits Workflow

Grits is an Agentic PM & OKR platform that automatically tracks AI agent work.
This skill guides you through the task tracking workflow using Grits MCP tools.

## Core Principle

**All non-trivial work must be recorded in Grits.** Register the task before modifying code.

Exceptions: simple Q&A, one-line fixes, read-only exploration.

## Standard Workflow

### 1. Check Current Status

```
task_context
```

Review tasks currently in progress and today's schedule. Determine if the user's request matches an existing task.

### 2. Start Work

**When an existing task matches:**
```
work_start(task_id: "<id>", description: "Describe what will be done")
```

**When it's new work:**
```
work_start(title: "Clear, actionable title", description: "What and why")
```

**Title rules:**
- Refine informal titles: "this looks like a good approach" → "Review prompt generation based on task context"
- If an existing task has a vague title, pass an improved one

**Description rules:**
- Always include a description. If an existing task has none, add one.

### 3. Check Linkage

The `work_start` response includes a linkage section:

```json
{
  "linkage": {
    "has_milestone": false,
    "has_kr": false,
    "available_milestones": [...],
    "available_krs": [...]
  }
}
```

- If `has_milestone` or `has_kr` is `false`, review the suggestions
- If an item has `recommended: true` and a reasonable `reason`, link it:
  ```
  milestone_link(task_id: "<id>", milestone_id: "<id>")
  kr_link(task_id: "<id>", kr_id: "<id>")
  ```
- Skip if nothing is relevant

### 4. Do the Work

Modify code, test, and verify. If you need to log intermediate progress:

```
task_comment(task_id: "<id>", message: "Progress update")
```

### 5. Complete the Work

```
task_done(task_id: "<id>", message: "Summary of what was done")
```

Or:

```
work_done(task_id: "<id>", message: "Summary of what was done")
```

## Hook Enforcement

The Grits plugin uses a **PreToolUse hook** to enforce task registration before code changes.

- If `work_start` hasn't been called before editing code files, a `[Grits]` message will appear
- The message may show a **recent task** from the previous session:
  - If your work is related to the recent task, **continue it** with `work_start(task_id: "<ID>")`
  - If you want to log additional context, use `task_comment` first, then `work_start`
  - Only create a **new task** if the work is clearly unrelated
- Config/docs files (`.md`, `.json`, `.yaml`, etc.) are exempt
- After calling `work_start`, the hook won't interrupt again

## Available Tools

| Tool                 | Purpose                                          |
| -------------------- | ------------------------------------------------ |
| `task_context`       | Current in-progress tasks + today's schedule     |
| `work_start`         | Start work (create or transition to IN_PROGRESS) |
| `work_done`          | Complete work + comment                          |
| `task_create`        | Create a task (without starting it)              |
| `task_batch_create`  | Create multiple tasks at once                    |
| `task_update`        | Change status/fields                             |
| `task_done`          | Mark as done                                     |
| `task_get`           | Get task details                                 |
| `task_list`          | List tasks                                       |
| `task_next`          | Get next recommended task                        |
| `task_comment`       | Add a comment                                    |
| `task_report`        | Generate a work report                           |
| `milestone_link`     | Link to a milestone                              |
| `kr_link`            | Link to a Key Result                             |
| `tag_list`           | List tags                                        |
| `tag_create`         | Create a tag                                     |
| `dashboard_insights` | Dashboard insights                               |
| `share_link_create`  | Create a share link                              |
