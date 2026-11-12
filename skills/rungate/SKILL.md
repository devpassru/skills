---
name: rungate
description: "Configure RunGate AI Gateway (OpenAI-compatible proxy at https://gateway.rungate.ru/v1 for Claude, GPT-4o, DeepSeek, and Gemini) and RunGate Smart DNS (DoH at https://dns.rungate.ru/dns-query). Use when integrating LLM SDKs via RunGate or configuring DNS-over-HTTPS for developer tools without VPN."
license: MIT
---

# RunGate AI Gateway & Smart DNS

## Overview

RunGate provides two capabilities under a unified DevPass token (`DEVPASS_TOKEN`):
1. **AI Gateway (`https://gateway.rungate.ru/v1`):** OpenAI-compatible HTTP proxy for Claude, GPT-4o, DeepSeek, and Gemini without VPN.
2. **Smart DNS (`https://dns.rungate.ru/dns-query`):** DNS-over-HTTPS (DoH) for accessing AI APIs and developer tools natively.

## When to Use

- Configuring OpenAI JS/TS or Python SDKs to route requests through RunGate AI Gateway.
- Calling Claude, GPT-4o, DeepSeek, or Gemini models using `DEVPASS_TOKEN`.
- Setting up RunGate Smart DNS (DoH) on macOS, Windows, Linux, iOS, Android, or routers.

## AI Gateway Integration

- **Base URL:** `https://gateway.rungate.ru/v1`
- **Auth header:** `Authorization: Bearer $DEVPASS_TOKEN`
- **Compatible models:** `claude-3-7-sonnet`, `claude-3-5-sonnet`, `gpt-4o`, `gpt-4o-mini`, `deepseek-v3`, `deepseek-r1`, `gemini-2.5-pro`, `gemini-2.5-flash`.

### JavaScript / TypeScript (OpenAI SDK)

```ts
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.DEVPASS_TOKEN,
  baseURL: "https://gateway.rungate.ru/v1",
});
```

### Python (OpenAI SDK)

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["DEVPASS_TOKEN"],
    base_url="https://gateway.rungate.ru/v1",
)
```

### cURL

```bash
curl https://gateway.rungate.ru/v1/chat/completions \
  -H "Authorization: Bearer $DEVPASS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-3-7-sonnet", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Smart DNS (DoH)

- **DoH URL:** `https://dns.rungate.ru/dns-query`
- **Android Private DNS hostname:** `dns.rungate.ru`
- **macOS profile:** `https://rungate.ru/download/macos.mobileconfig`

## MCP Tools (`@devpass/mcp`)

- `rungate_get_gateway_config` — returns active `baseUrl`, masked token check, compatible models, and SDK snippets.
- `rungate_check_dns` — returns OS-specific Smart DNS setup steps (`macos`, `windows`, `linux`, `ios`, `android`, `router`).

## Common Pitfalls

- **Exposing `DEVPASS_TOKEN` in client-side code:** Only call `gateway.rungate.ru/v1` from backend services, scripts, or local dev tools—never embed `DEVPASS_TOKEN` in browser bundles.
