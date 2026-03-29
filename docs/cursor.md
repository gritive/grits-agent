# Grits + Cursor

## Setup

Add to your Cursor MCP settings (`.cursor/mcp.json`):

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

## Prerequisites

1. Install: `brew tap gritive/tap && brew install grits`
2. Authenticate: `grits login`
3. Restart Cursor after adding MCP configuration
