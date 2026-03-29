# Grits MCP Server

Grits CLI includes a built-in MCP (Model Context Protocol) server. Any MCP-compatible AI agent can connect to Grits.

## How It Works

The MCP server runs as a subprocess, communicating via JSON-RPC over stdin/stdout:

```bash
grits mcp
```

## Available Tools

| Tool | Description |
|------|-------------|
| `task_next` | Get next prioritized task |
| `task_context` | Current work context (in-progress + today's schedule) |
| `task_list` | List tasks with filters |
| `task_get` | Get task details |
| `task_create` | Create a new task |
| `task_update` | Update task status/fields |
| `task_done` | Mark task as complete |
| `task_comment` | Add comment to task |
| `task_report` | Task summary report |
| `task_batch_create` | Create multiple tasks at once |
| `work_start` | Start working on a task (high-level) |
| `work_done` | Finish current work (high-level) |
| `dashboard_insights` | Dashboard summary |
| `tag_list` | List tags |
| `tag_create` | Create a tag |
| `milestone_link` | Link task to milestone |
| `kr_link` | Link task to Key Result |
| `share_link_create` | Create a share link |

## Prerequisites

1. Install Grits CLI: `brew tap gritive/tap && brew install grits`
2. Authenticate: `grits login`

## Configuration

Add to your AI agent's MCP configuration:

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

See agent-specific guides for detailed setup.
