# Grits + Claude Code

## Setup

### Option 1: Claude Code Plugin (Recommended)

```bash
# Register from marketplace (one-time)
claude plugin marketplace add gritive/grits-agent

# Install plugin
claude plugin install grits
```

This installs everything at once:
- **MCP Server** — `grits mcp` for task/OKR management
- **Hooks** — PreToolUse (work registration), PostToolUse (commit auto-comment), Stop (session end cleanup)
- **Skills** — `/start` (start work workflow), `/done` (complete work workflow), `/workflow` (Grits guide)

### Option 2: Manual MCP Configuration

Add to your Claude Code settings (`.claude/settings.json` or project-level):

```json
{
  "mcpServers": {
    "grits": {
      "command": "grits",
      "args": ["mcp"]
    }
  }
}
```

> Note: Manual configuration only sets up the MCP server. Hooks and skills require the plugin.

### Option 3: Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "grits": {
      "command": "grits",
      "args": ["mcp"]
    }
  }
}
```

## What the Plugin Does

### Hooks

| Hook | Trigger | Behavior |
| --- | --- | --- |
| **PreToolUse** | Edit / Write / MultiEdit | Reminds agent to call `work_start` before modifying code files. Config/docs files (`.md`, `.json`, `.yaml`, etc.) are exempt. |
| **PostToolUse** | Bash (git commit) | Auto-comments the commit message on the linked Grits task. Requires branch name `grits/<TASK_ID>`. |
| **Stop** | Session end | Shows active task info and cleans up the work-active marker. |

All hooks use `grits hook` subcommands — no external dependencies (python, jq, etc.).

### Skills

| Skill | Trigger | Description |
| --- | --- | --- |
| `/start` | "start work", "what's next" | Check status → select task → write description → work_start → implement |
| `/done` | "task complete", "done" | Verify → commit → task_done → suggest next |
| `/workflow` | "grits workflow" | Grits workflow guide |

### MCP Tools

28 tools for task management, OKR tracking, and work lifecycle. See [MCP docs](mcp.md) for the full list.

## Project Scoping

Run `grits init` in your project directory to scope MCP tools to a specific Grits project:

```bash
grits init                      # List projects
grits init --project <ID>       # Create .grits.json
```

This creates `.grits.json` in the project root. MCP tools (`task_context`, `task_next`, `task_list`, `work_start`, etc.) will automatically filter to this project.

## Usage

Once connected, Claude can:

- Check your current tasks: *"What should I work on next?"*
- Start work: Automatically calls `work_start` before coding
- Complete tasks: Calls `work_done` when finished
- Create tasks: *"Create a task for fixing the login bug"*
- Track progress: *"Show me the task report"*
- Link to OKR: *"Link this task to the KR"*

## Workflow Integration

Add to your project's `CLAUDE.md`:

```markdown
## Grits Integration
- Before starting work, check `task_context` for current tasks
- Use `work_start` when beginning a task, `work_done` when completing
- All non-trivial work must be tracked in Grits
```
