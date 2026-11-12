---
name: devpass
description: "Configure and use the DevPass developer ecosystem (@devpass/mcp, DEVPASS_TOKEN, and account quotas). Use when setting up DevPass MCP in Cursor, Claude Code, or Windsurf, authenticating with a dp_live_ token, or checking subscription limits across Submito, RunDeploy, RunPing, RunGate, RunPort, Adopto, and Answero."
license: MIT
---

# DevPass Ecosystem & MCP Setup

## Overview

DevPass (`https://devpass.ru`) provides a unified subscription, SSO (`https://id.devpass.ru`), and API token (`dp_live_...`) for seven developer infrastructure products, accessible through the `@devpass/mcp` server.

## When to Use

- User wants to connect `@devpass/mcp` to Cursor, Claude Code, Windsurf, or another MCP client.
- User needs to authenticate an agent or script using `DEVPASS_TOKEN` (`dp_live_...`).
- User asks to check their DevPass plan, service quotas, or available ecosystem tools.

## Setup `@devpass/mcp`

### 1. Obtain `DEVPASS_TOKEN`

Direct the user to sign in at `https://id.devpass.ru` and copy their master API token (`dp_live_...`). Never hardcode `dp_live_...` tokens in source code or commit them to Git.

### 2. Configure the MCP Client

**Claude Code:**
```bash
claude mcp add devpass npx -y @devpass/mcp --env DEVPASS_TOKEN="dp_live_your_token_here"
```

**Cursor (`.cursor/mcp.json`) / Windsurf (`~/.codeium/windsurf/mcp_config.json`):**
```json
{
  "mcpServers": {
    "devpass": {
      "command": "npx",
      "args": ["-y", "@devpass/mcp"],
      "env": {
        "DEVPASS_TOKEN": "dp_live_your_token_here"
      }
    }
  }
}
```

## Checking Account Status

If the `devpass` MCP server is connected, call `devpass_status` to inspect the active plan (`Free` / `DevPass Pro`) and per-service quotas before provisioning resources.

## Ecosystem Products

| Service | Purpose | Endpoint / Entry |
| :--- | :--- | :--- |
| **RunDeploy** | Static/SPA/Telegram Mini App hosting & atomic releases | `https://rundeploy.ru` |
| **Submito** | Agent-friendly forms & lead intake backend | `https://app.submito.ru` |
| **RunGate** | OpenAI-compatible AI Gateway & Smart DNS (DoH) | `https://gateway.rungate.ru/v1` |
| **RunPing** | Uptime, SSL, and public status pages | `https://runping.ru` |
| **RunPort** | Reverse SSH tunnels to expose `localhost` over HTTPS | `srv.runport.ru:2222` |
| **Adopto** | Behavioral event tracking & email sequences | `https://api.adopto.ru` |
| **Answero** | AI + Telegram support chat widget | `https://answero.ru` |

## Common Pitfalls

- **Committing tokens:** Always store `DEVPASS_TOKEN` in environment variables or `.env` (gitignored), never in frontend bundles or committed config files.
- **Missing token in MCP:** If any `run*` / `submito_*` / `adopto_*` / `answero_*` tool returns `DEVPASS_TOKEN is not set`, verify the `env` block in the MCP configuration and restart the MCP client.
