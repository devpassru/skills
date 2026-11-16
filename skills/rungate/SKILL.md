---
name: rungate
description: "Expert guide for RunGate AI Gateway (gateway.rungate.ru/v1) and Smart DNS (dns.rungate.ru). Use when choosing between AI Gateway and Smart DNS, integrating OpenAI/Python/TS SDKs with ruble billing, selecting models (Claude, GPT, Gemini, DeepSeek, Qwen, Kimi), setting up DoH/DoT on macOS, Windows, Linux, iOS, Android, or routers without VPN, or debugging DNS/gateway issues."
license: MIT
---

# RunGate AI Gateway & Smart DNS Expert

You are an expert on **RunGate** (`https://rungate.ru`), which provides two distinct tools for accessing global AI APIs and developer services without VPN:

## 1. AI Gateway vs. Smart DNS — Which One Does the User Need?

Diagnose the user's goal first:

| User Need | Use | Why |
| :--- | :--- | :--- |
| **Build an app, bot, or script calling LLMs** | **AI Gateway** (`https://gateway.rungate.ru/v1`) | Single OpenAI-compatible API, ruble top-up via DevPass ID, per-key spending caps (`limitRub`), token/cost telemetry |
| **Use Cursor, Copilot, Antigravity, Claude Desktop, ChatGPT Web, or native Anthropic SDK** | **Smart DNS** (`https://dns.rungate.ru/dns-query`) | Zero-TLS-decryption DoH + SNI proxy routes only AI domains through `88.218.171.12` while Russian banks, Gosuslugi, and local sites go direct |
| **Both** | Configure both | Gateway for production backend code; Smart DNS on dev machine/router for IDE and browser tools |

## 2. AI Gateway (`https://gateway.rungate.ru/v1`)

### Auth & Billing Model
- **Control Plane (`https://app.rungate.ru/api/*`):** Uses `DEVPASS_TOKEN` (`dp_live_...`) to manage keys and check billing.
- **Inference Plane (`https://gateway.rungate.ru/v1`):** Uses a **`sk-...` Gateway key** (48 hex chars), created via `rungate_create_gateway_key` or `POST /api/gateway/keys` with optional `limitRub`. **Never** pass `dp_live_...` to the gateway or embed `sk-...` in browser bundles.
- **Billing:** Pay-as-you-go in rubles (`1 RUB = 5,000 Quota`). Top up via DevPass ID (`https://id.devpass.ru`, min 100 RUB). Streaming (`stream: true` SSE) is supported across all models.

### Model Selection Matrix (Configured Models)
- **Flagship reasoning & coding:** `claude-opus-5-5` (600/3000 RUB per 1M in/out, 1M ctx), `claude-sonnet-5-5` (350/1750 RUB, 1M), `gpt-6.1-sol` (300/1500 RUB, 1M), `gemini-4-argon` (300/1500 RUB, 1M)
- **Fast & cost-effective:** `claude-haiku-5-5` (95/475 RUB, 1M), `gpt-6.1-terra` (150/750 RUB, 1M), `gemini-3.8-flash` (110/550 RUB, 1M)
- **Budget / high-volume / coding:** `deepseek-v4.1-flash` (25/85 RUB, 1M), `deepseek-chat` (45/135 RUB, 128k), `deepseek-v4-pro` (65/220 RUB, 1M), `glm-5.3` (90/360 RUB, 1M), `kimi-for-coding` (180/900 RUB, 1M), `qwen4-max` (220/660 RUB, 1M)

### SDK Integration
```ts
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.RUNGATE_GATEWAY_KEY, // sk-...
  baseURL: "https://gateway.rungate.ru/v1",
});
```
```python
import os
from openai import OpenAI

client = OpenAI(
    api_key=os.environ["RUNGATE_GATEWAY_KEY"],
    base_url="https://gateway.rungate.ru/v1",
)
```
> **Note:** OpenAI SDK works 100% for all models (including Claude) via `/v1/chat/completions`. If the user specifically needs the **official Anthropic SDK** talking directly to `api.anthropic.com`, use **Smart DNS** instead.

## 3. Smart DNS (DoH + Zero-TLS-Decryption SNI Proxy)

### How It Works
1. `ai-doh-server` (`dns.rungate.ru`) validates the user's profile `<token>` and registers the client's IPv4 in the dynamic allowlist (`88.218.171.12`).
2. Only AI/dev domains (OpenAI/ChatGPT/Sora, Anthropic/Claude, GitHub Copilot, Google Gemini/AI Studio/Antigravity, Cursor/Anysphere, v0, Groq, OpenRouter, Mistral, DeepSeek, Perplexity, xAI, Fireworks, Together, Replicate) resolve to the SNI proxy; everything else resolves normally via public DNS.
3. `ai-sni-proxy` forwards TLS based on `ClientHello` SNI—**no MITM certificates, no TLS decryption**.
4. **Device Limits:** 24h free trial (1 profile via `POST /api/dns/activate-trial`), Developer = 1 profile, Power = 5, Team = 20.

### OS & Router Setup
- **macOS / iOS (Recommended):** Download the `.mobileconfig` profile from `https://dns.rungate.ru/mobileconfig/<token>` and install under *System Settings -> VPN & Device Management* (or use `DNSSwitcher` on macOS).
- **Windows 11:** Network Adapter -> IPv4 DNS Manual -> Encrypted only (DNS over HTTPS) -> template `https://dns.rungate.ru/dns-query/<token>`.
- **Linux (`systemd-resolved`):** Set `DNS=185.130.44.155#dns.rungate.ru` and `DNSOverTLS=yes` in `/etc/systemd/resolved.conf`, then `sudo systemctl restart systemd-resolved` (or run `cloudflared` DoH proxy).
- **Android:** Settings -> Network -> Private DNS -> Hostname: `dns.rungate.ru`.
- **Routers (Keenetic / OpenWrt):** Add DoH upstream `https://dns.rungate.ru/dns-query/<token>` to cover the whole network.
- **Browsers (Chrome/Edge/Brave/Firefox):** Security -> Use secure DNS -> Custom -> `https://dns.rungate.ru/dns-query/<token>`.

## 4. Verification & Troubleshooting

- **Check DoH server health:** `curl -i https://dns.rungate.ru/health` (expect `200 OK`, `healthy`).
- **Verify DoH resolution for an AI domain:**
  ```bash
  curl -H "accept: application/dns-message" \
    "https://dns.rungate.ru/dns-query/<token>?dns=q80BAAABAAAAAAAAA3d3dwdvcGVuYWkDY29tAAABAAE"
  ```
- **Test Gateway key & model:**
  ```bash
  curl https://gateway.rungate.ru/v1/chat/completions \
    -H "Authorization: Bearer $RUNGATE_GATEWAY_KEY" \
    -H "Content-Type: application/json" \
    -d '{"model": "claude-sonnet-5-5", "messages": [{"role": "user", "content": "ping"}]}'
  ```
- **Smart DNS stops working after IP change / Wi-Fi switch:** The client must send a DoH query through `dns.rungate.ru` so `ai-doh-server` refreshes the client IP in the SNI proxy allowlist. Flush local DNS cache (`sudo dscacheutil -flushcache` on macOS) or toggle Wi-Fi if stale A records are cached.
- **MCP tools (`@devpass/mcp`):** `rungate_get_gateway_config`, `rungate_list_gateway_keys`, `rungate_create_gateway_key`, `rungate_get_billing`, `rungate_list_dns_profiles`, `rungate_create_dns_profile`, `rungate_check_dns`.
