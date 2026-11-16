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

- **Control Plane (`https://app.rungate.ru/api/*`):** Authenticated with `Authorization: Bearer $DEVPASS_TOKEN` (`dp_live_...`). Use it (or `rungate_create_gateway_key` in MCP) to create a Gateway API key (`sk-...`).
- **Gateway Base URL:** `https://gateway.rungate.ru/v1`
- **Auth header:** `Authorization: Bearer $RUNGATE_GATEWAY_KEY` (the `sk-...` key created via RunGate API / `rungate_create_gateway_key`)
- **Compatible models:** `claude-sonnet-5-5`, `claude-opus-5-5`, `claude-haiku-5-5`, `gpt-6.1-sol`, `gpt-6.1-terra`, `gemini-4-argon`, `gemini-3.8-flash`, `deepseek-chat`, `deepseek-v4.1-flash`, `deepseek-v4-pro`, `qwen4-max`, `kimi-for-coding`, `glm-5.3`.

### JavaScript / TypeScript (OpenAI SDK)

```ts
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.RUNGATE_GATEWAY_KEY,
  baseURL: "https://gateway.rungate.ru/v1",
});
```

### Python (OpenAI SDK)

```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["RUNGATE_GATEWAY_KEY"],
    base_url="https://gateway.rungate.ru/v1",
)
```

### cURL

```bash
curl https://gateway.rungate.ru/v1/chat/completions \
  -H "Authorization: Bearer $RUNGATE_GATEWAY_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-sonnet-5-5", "messages": [{"role": "user", "content": "Hello!"}]}'
```

## Smart DNS (DoH)

- **DoH URL:** `https://dns.rungate.ru/dns-query`
- **Android Private DNS hostname:** `dns.rungate.ru`
- **macOS profile:** `https://rungate.ru/download/macos.mobileconfig`

## MCP Tools (`@devpass/mcp`)

- `rungate_list_gateway_keys` — list RunGate AI Gateway API keys (`DEVPASS_TOKEN`).
- `rungate_create_gateway_key` — create a new `sk-...` Gateway key (`DEVPASS_TOKEN`).
- `rungate_list_dns_profiles` — list Smart DNS (DoH) profiles (`DEVPASS_TOKEN`).
- `rungate_create_dns_profile` — create a new Smart DNS (DoH) profile (`DEVPASS_TOKEN`).
- `rungate_get_billing` — get AI Gateway balance, usage, and subscription summary (`DEVPASS_TOKEN`).
- `rungate_get_gateway_config` — returns active `baseUrl`, compatible models, and SDK snippets.
- `rungate_check_dns` — returns OS-specific Smart DNS setup steps (`macos`, `windows`, `linux`, `ios`, `android`, `router`).

## Common Pitfalls

- **Exposing `DEVPASS_TOKEN` in client-side code:** Only call `gateway.rungate.ru/v1` from backend services, scripts, or local dev tools—never embed `DEVPASS_TOKEN` in browser bundles.
