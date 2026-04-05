# Grits Agent

**Agentic PM for your terminal** — auto-track tasks, align work to OKR, get smart next-task recommendations.

[Grits](https://grits.gritive.com) turns Claude Code into a project-aware agent. Write code, and Grits tracks what you did, links it to your OKR, and tells you what to do next.

> **🌱 Early Access — 10 teams** We're looking for 10 AI-native dev teams to join our founding cohort. Free forever.
> → [Apply at grits.gritive.com/signup](https://grits.gritive.com/signup)

## What It Does

- **Auto-tracking**: Code changes are automatically linked to tasks via hooks — no manual updates
- **Smart start**: `/grits:start` scans your backlog, today's schedule, and code TODOs to recommend your next high-impact task
- **OKR alignment**: Tasks auto-link to milestones and key results, so daily work connects to strategy
- **Completion flow**: `/grits:done` validates, commits, updates Grits, and suggests what's next

## Installation

### 1. Install CLI

```bash
brew tap gritive/tap && brew install grits
```

### 2. Authenticate

```bash
grits auth login
```

### 3. Install Claude Code Plugin

```bash
claude plugin install grits-agent/grits
```

## Quick Start

After installing, just start coding. The plugin hooks will remind you to register work when you edit files.

Or explicitly:

```
/grits:start          # See what to work on next
/grits:done           # Finish current task with validation
/grits:workflow       # Learn the full workflow
```

### CLI Examples

```bash
grits task list -o table              # List tasks (human-readable)
grits task get <ID> -o table          # Task detail card
grits task create --title "Fix bug"   # Create a task
grits task done <ID>                  # Mark complete
grits dashboard summary               # Work summary
```

## How It Works

```
You code ──→ Hook detects edit ──→ Grits asks "what are you working on?"
                                          │
                                    work_start()
                                          │
                                    Task → IN_PROGRESS
                                    Link → Milestone + KR
                                          │
You finish ──→ /grits:done ──→ Validate → Commit → task_done()
                                          │
                                    Suggest next task
```

**3 Hooks** keep you on track:
- `PreToolUse` — reminds to register work before code edits
- `PostToolUse` — auto-comments commits to linked tasks
- `Stop` — session summary on exit

**3 Skills** guide the workflow:
- `/grits:start` — find and start the right task
- `/grits:done` — validate, commit, complete
- `/grits:workflow` — reference guide

## MCP Server

Grits CLI includes a built-in MCP server with 28 tools for AI agent integration.

```bash
grits mcp   # Start MCP server (stdin/stdout)
```

Setup guides for other editors:
- [Cursor](docs/cursor.md)
- [Windsurf](docs/windsurf.md)
- [General MCP](docs/mcp.md)

## Documentation

- [CLI Reference](docs/cli.md) — all commands and flags
- [MCP Server](docs/mcp.md) — tool list and configuration
- [Claude Code Setup](docs/claude-code.md) — detailed integration guide

## Early Access

We're actively recruiting 10 AI-native development teams for our founding cohort.

**What you get:**
- Free forever (no credit card, no catch)
- Direct line to the founding team — your feedback shapes the roadmap
- Early adopter badge on your team profile

**Who we're looking for:** Teams that use Claude Code, Cursor, or Windsurf daily and want PM tooling that lives in the terminal.

[Sign up at grits.gritive.com](https://grits.gritive.com/signup) — 10 spots, first come first served.

---

If this is useful, a ⭐ helps others find it.
