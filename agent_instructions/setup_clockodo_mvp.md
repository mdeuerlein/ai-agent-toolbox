# Clockodo MCP – Autonomous Setup Instructions

These instructions are written for an AI agent. Read them completely before taking any action, then execute each step in order without waiting for confirmation unless a step explicitly requires user input.

---

## What you are setting up

You are connecting the **Clockodo time tracking service** to this environment via the **Model Context Protocol (MCP)**. After completing this setup, the user will be able to interact with their Clockodo data (time entries, projects, clients, etc.) through you.

---

## Step 1 – Determine the correct MCP config location

Identify which agent runtime or MCP host is active in this environment. Common cases:

| Environment | Config file location |
|---|---|
| Claude Desktop | `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows) |
| Cursor / VS Code (Cline, Continue, etc.) | `.cursor/mcp.json` or `.vscode/mcp.json` in the workspace root, or the global settings file of the respective extension |
| OpenClaw / Hermes Agent | Check the agent's own documentation or config directory for the MCP server configuration file |
| OpenAI Codex | `~/.codex/config.json` or the `mcpServers` block in the project-level `codex.yaml` / `codex.json` if present |
| Other | Look for any existing `mcp.json`, `mcp_servers.json`, or `mcpServers` block in the agent's config |

> **Note:** If the information in this table conflicts with the agent's own local settings, documentation, or built-in defaults, the agent's local configuration takes precedence over this table.

If you cannot determine the location automatically, **stop here and ask the user** where their MCP configuration file is located.

---

## Step 2 – Request the API key from the user

Before writing any configuration, ask the user for their Clockodo API key with this exact message:

> "To connect Clockodo, I need your personal Clockodo API key. Please retrieve it from **Clockodo → My Data → API** and store it in a secure location appropriate for this system — for example, an environment variable, a secrets manager, or your system's credential store. Do not paste it directly into the chat. Once it is stored, let me know the variable name or path so I can reference it in the configuration."

Wait for the user's response before continuing. Do not proceed with a placeholder key.

---

## Step 3 – Write the MCP server configuration

Add the following block to the MCP configuration file identified in Step 1. If the file already contains an `mcpServers` object, add the `clockodo` entry to it. If the file does not exist yet, create it with the full structure shown below.

```json
{
  "mcpServers": {
    "clockodo": {
      "type": "http",
      "url": "https://mcp.clockodo.com/mcp",
      "description": "Clockodo time tracking MCP server",
      "headers": {
        "X-Clockodo-User": "CLOCKODO_USER_EMAIL",
        "X-Clockodo-Key": "CLOCKODO_API_KEY"
      }
    }
  }
}
```

Replace `CLOCKODO_USER_EMAIL` with the user's Clockodo login email address and `CLOCKODO_API_KEY` with the API key reference obtained in Step 2 (e.g. an environment variable reference like `$CLOCKODO_API_KEY` if the runtime supports it, or the key value itself if the user has explicitly confirmed that direct storage is acceptable for their security requirements).

---

## Step 4 – Verify the configuration

After writing the config file:

1. Confirm to the user that the configuration has been written and show the file path.
2. If the runtime requires a restart to pick up new MCP servers, tell the user: *"Please restart [runtime name] for the Clockodo MCP server to become active."*
3. Once restarted (or immediately if no restart is needed), attempt a simple read-only call to the Clockodo MCP server — for example, fetching the current user's profile or listing available projects — to confirm the connection works.
4. Report the result to the user: either confirm success and give a short example of what data is now accessible, or report the error and suggest a fix.

---

## Important notes

- **Write access:** The Clockodo MCP server supports tools with the prefixes `create`, `update`, and `delete`. These can modify or delete data in the user's Clockodo account. Only enable these tools if the user has explicitly asked for write access. For read-only setups, restrict tool access to `get` and `list` prefixed tools only.
- **Credentials:** Never log or display the API key in full after the initial setup. If you need to reference it in output, mask it (e.g. `sk-****`).
- **MCP server URL:** `https://mcp.clockodo.com/mcp`
- **Authentication method:** HTTP headers (`X-Clockodo-User` + `X-Clockodo-Key`). Basic Auth is also supported if the runtime requires it — refer to the Clockodo API documentation for the Basic Auth format.
