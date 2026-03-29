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
grits auth login                # Authenticate with Grits (browser-based)
grits auth login --token <PAT>  # Authenticate with PAT
grits auth list                 # List profiles
grits auth use <name>           # Switch profile
grits auth me                   # Show current user
grits auth logout [profile]     # Logout
grits auth workspace            # Show current workspace
```

### Tasks

```bash
grits task list                 # List tasks
grits task list --project <id>  # List tasks in project
grits task get <id>             # Get task details
grits task create --title "..." # Create task
grits task update <id> --status IN_PROGRESS
grits task done <id>            # Mark task complete
grits task move <id> [parent]   # Move task
grits task delete <id>          # Delete task
grits task search <query>       # Search tasks
grits task next                 # Next prioritized task
grits task context              # In-progress + today's schedule
grits task report               # Task report
grits task batch-create         # Batch create tasks
grits task comment <id> <msg>   # Add comment
```

### Dashboard

```bash
grits dashboard summary         # Work summary
grits dashboard insights        # AI insights
```

### Projects

```bash
grits project list              # List projects
grits project get <id>          # Get project details
grits project create            # Create project
grits project update <id>       # Update project
grits project delete <id>       # Delete project
```

### OKR

```bash
grits okr cycle list            # List OKR cycles
grits okr cycle get <id>        # Get cycle details
grits okr cycle create          # Create cycle
grits okr dashboard <cycle-id>  # OKR dashboard
grits okr objective list        # List objectives
grits okr objective get <id>    # Get objective
grits okr objective create      # Create objective
grits okr kr list --objective <id>  # List key results
grits okr kr update <id>        # Update KR
grits okr kr link <kr-id> <task-id>    # Link task to KR
grits okr kr unlink <kr-id> <task-id>  # Unlink task from KR
grits okr kr tasks <kr-id>      # List tasks linked to KR
```

### Comments

```bash
grits comment list <task-id>    # List comments
grits comment add <task-id> <content>  # Add comment
```

### Tags

```bash
grits tag list                  # List tags
grits tag create --name "..."   # Create tag
```

### Configuration

```bash
grits config show               # Show config
grits config get <key>          # Get config value
grits config set <key> <value>  # Set config value
grits config path               # Show config file path
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
grits status list               # List statuses
grits workspace list            # List workspaces
grits workspace members         # List workspace members
grits notification list         # List notifications
grits notification read <id>    # Mark notification as read
grits share-link create <id>    # Create share link
grits share-link list <id>      # List share links
```
