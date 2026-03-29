# Grits CLI Reference

## Global Flags

| Flag          | Short | Description                                      |
| ------------- | ----- | ------------------------------------------------ |
| `--output`    | `-o`  | Output format: json, table, yaml (default: json) |
| `--profile`   | `-p`  | Use a specific profile                           |
| `--server`    |       | Override server URL                              |
| `--workspace` | `-w`  | Override workspace ID                            |
| `--verbose`   | `-v`  | Show request/response details                    |
| `--field`     | `-f`  | Extract a single field value                     |

## Commands

### Authentication

```bash
grits auth login                     # Authenticate with Grits
grits config list               # List profiles
grits config set-profile <name> # Switch profile
```

### Tasks

```bash
grits next                      # Next prioritized task
grits context                   # In-progress + today's schedule
grits task list                 # List tasks
grits task list --project <id>  # List tasks in project
grits task create --title "..." # Create task
grits task get <id>             # Get task details
grits task update <id> --status IN_PROGRESS
grits done <id>                 # Mark task complete
```

### Projects

```bash
grits project list              # List projects
```

### OKR

```bash
grits okr cycles                # List OKR cycles
grits okr dashboard <cycle-id>  # OKR dashboard
```

### MCP Server

```bash
grits mcp                       # Start MCP server (stdin/stdout)
```

### Hook Helpers

Used by Claude Code plugin hooks. Not intended for direct use.

```bash
grits hook pre-edit             # Check work registration before code edits (stdin: TOOL_INPUT)
grits hook post-commit          # Auto-comment Grits task after git commit (stdin: TOOL_INPUT)
grits hook session-end          # Show active task info + cleanup on session end
```

### Other

```bash
grits version                   # Show version
grits tag list                  # List tags
grits tag create --name "..."   # Create tag
```
