# Grits + Claude Code

## Setup

### Option 1: Claude Code Plugin (Recommended)

```bash
claude plugin add github:gritive/grits-agent/claude-plugin
```

This installs everything at once:
- **MCP Server** — `grits mcp` for task/OKR management
- **Hooks** — PreToolUse (work registration), PostToolUse (commit auto-comment), Stop (session end cleanup)
- **Skills** — `/start` (작업 시작 워크플로우), `/done` (작업 완료 워크플로우), `/workflow` (Grits 가이드)

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
| `/start` | "작업 시작", "다음 할 일" | 현황 파악 → 태스크 선택 → 설명 작성 → work_start → 구현 |
| `/done` | "작업 완료", "끝" | 검증 → 커밋 → task_done → 다음 추천 |
| `/workflow` | "grits workflow" | Grits 워크플로우 가이드 |

### MCP Tools

28 tools for task management, OKR tracking, and work lifecycle. See [MCP docs](mcp.md) for the full list.

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
