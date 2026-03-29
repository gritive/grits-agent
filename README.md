# Grits Agent

CLI & MCP Server for [Grits](https://grits.so) — Agentic PM & OKR platform.

Manage tasks, track OKR progress, and connect AI agents to Grits without leaving your terminal.

## Installation

### Homebrew (macOS & Linux)

```bash
brew tap gritive/tap
brew install grits
```

### Manual Download

Download the latest binary from [Releases](https://github.com/gritive/grits-agent/releases).

## Quick Start

### 1. Authenticate

```bash
grits login
```

### 2. Use CLI

```bash
grits next              # Next task to work on
grits task list         # List my tasks
grits done <TASK_ID>    # Complete a task
grits context           # Current work context
```

### 3. Connect as MCP Server

Grits CLI includes a built-in MCP (Model Context Protocol) server for AI agent integration.

See setup guides:
- [Claude Code / Claude Desktop](docs/claude-code.md)
- [Cursor](docs/cursor.md)
- [Windsurf](docs/windsurf.md)
- [General MCP Setup](docs/mcp.md)

## Documentation

- [CLI Reference](docs/cli.md)
- [MCP Server](docs/mcp.md)

## Claude Code Plugin

See [`claude-plugin/`](claude-plugin/) for the Claude Code plugin configuration.
