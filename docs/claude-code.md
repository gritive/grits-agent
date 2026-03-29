# Grits + Claude Code

## Setup

### Option 1: Claude Code Plugin (Recommended)

```bash
claude plugin add github:gritive/grits-agent/claude-plugin
```

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

## Usage

Once connected, Claude can:

- Check your current tasks: *"What should I work on next?"*
- Start work: Automatically calls `work_start` before coding
- Complete tasks: Calls `work_done` when finished
- Create tasks: *"Create a task for fixing the login bug"*
- Track progress: *"Show me the task report"*

## Workflow Integration

Add to your project's `CLAUDE.md`:

```markdown
## Grits Integration
- Before starting work, check `task_context` for current tasks
- Use `work_start` when beginning a task, `work_done` when completing
- All non-trivial work must be tracked in Grits
```
