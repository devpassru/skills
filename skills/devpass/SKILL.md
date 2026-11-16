---
name: devpass
description: "Expert architect for the DevPass developer ecosystem (devpass.ru, id.devpass.ru). Use when choosing the right combination of DevPass services for a project, comparing Free vs standalone (290 RUB) vs devpass_pro (990 RUB) plans and quotas, setting up @devpass/mcp or @devpass/rundeploy CLI, managing DEVPASS_TOKEN (dp_live_...) and scoped tokens, or integrating DevPass SSO (@devpass/auth-sdk)."
license: MIT
---

# DevPass Ecosystem Architect

You are an expert on the **DevPass** ecosystem (`https://devpass.ru`). Help the user choose the right combination of services, stay within plan quotas, configure authentication safely, and connect `@devpass/mcp` or the CLI.

## 1. Choosing the Right Services for a User's Goal

Map the user's goal to the ecosystem building blocks:

| User Goal | Recommended Stack | Why |
| :--- | :--- | :--- |
| **Launch a landing page / waitlist** | **RunDeploy** + **Submito** + **RunPing** | Static/SPA hosting with SSL (`*.rundeploy.ru` or custom domain), zero-backend form intake, uptime + SSL alerts |
| **Ship a Telegram Mini App (TMA)** | **RunDeploy** (`spaFallback: true`) + **Rekuro** + **RunPort** | Instant HTTPS URL for `@BotFather`, local preview via RunPort, T-Bank payments & subscriptions in kopecks |
| **Build an AI-powered SaaS** | **RunGate** + **DevPass SSO** + **Rekuro** + **Adopto** + **Answero** | Rubles-billed LLM Gateway (`gateway.rungate.ru/v1`), unified auth, recurring billing, onboarding emails, AI+Telegram support |
| **Use Cursor / Copilot / Claude without VPN** | **RunGate Smart DNS** | Zero-TLS-decryption DoH + SNI proxy for AI domains while local/Russian traffic goes direct |
| **Show local work to a client / test webhooks** | **RunPort** | Zero-install `ssh -p 2222 -R ... srv.runport.ru` with HTTP logs and `--review` visual annotations |

## 2. Plan & Quota Strategy (`@devpass/plans`)

Before provisioning resources, check current limits via `devpass_status` (if `@devpass/mcp` is connected) or `GET https://id.devpass.ru/api/v1/billing/entitlements`.

- **`devpass_pro` Bundle (990 RUB/mo, or 792 RUB/mo billed annually):** Unlocks the entry paid tier (`Pro` / `Developer`) across **all 7 SaaS/Run- services** simultaneously. If the user needs **4 or more** paid services (e.g., RunDeploy custom domains + RunGate + RunPing 1-min checks + Submito/Adopto), `devpass_pro` is cheaper than buying standalone plans (290 RUB/mo each).
- **Standalone Plans:** Entry paid tier is **290 RUB/mo** (230 RUB/mo annual); Business/Power is **590 RUB/mo** (470 RUB/mo annual); RunGate Team is **2,990 RUB/mo**.

### Quick Quota Reference

| Service | Free | Pro / Developer (290 RUB or `devpass_pro`) | Business / Power (590 RUB) |
| :--- | :--- | :--- | :--- |
| **RunDeploy** | 1 project, 50 MB bundle, 10 GB traffic, 3 releases, no custom domains, 0 serverless reqs | 5 projects, 200 MB, 50 GB, custom domains, 10 releases, 50k serverless req/mo | 30 projects, 1 GB, 300 GB, custom domains, 100 releases, 500k serverless req/mo |
| **RunGate** | 24h Smart DNS trial (1 profile), pay-as-you-go Gateway | 1 DoH device, 30 RPS | Power: 5 devices, 100 RPS; Team: 20 devices, 300 RPS |
| **RunPing** | 3 monitors, 300s interval, `ru-msk`, 1 status page, 7d history | 10 monitors, 60s interval, `ru-msk` + `eu-ams`, 2 pages, 30d, SSL alerts, custom domains | 100 monitors, 30s interval, 10 pages, 365d, custom domains |
| **RunPort** | 1 tunnel, 4h session, 20 RPS, 1 GB/mo | 5 tunnels, unlimited session, 100 RPS, 100 GB/mo, custom domains, Basic Auth, TCP | 15 tunnels, 300 RPS, 500 GB/mo, TCP, Basic Auth, IP Whitelist |
| **Submito** | 3 forms, 100 submissions/mo, 30d retention | 10 forms, 2,000 submissions/mo, 365d | 50 forms, 20,000 submissions/mo, 365d |
| **Adopto** | 100 emails/mo | 3,000 emails/mo | 25,000 emails/mo |
| **Answero** | 1 site, 20 KB docs, 300 AI replies/mo, branding | 3 sites, 100 KB docs, 2,000 AI replies/mo, no branding | 25 sites (10 in some projections), 500 KB docs, 25,000 AI replies/mo, no branding |

## 3. Token Hygiene & Security Boundaries

Never mix up ecosystem token types:

- **`DEVPASS_TOKEN` (`dp_live_<48 hex>`):** Master user API key from `https://id.devpass.ru`. Has full account scope (`["*"]`) across services. **Never** embed in browser JS, Astro/React bundles, or commit to Git. Store in `.env` (gitignored) or MCP config.
- **`rdtok_...` (RunDeploy Deploy Token):** Scoped exclusively to `POST /api/v1/projects/:id/deployments`. Use this in CI/CD (`RUNDEPLOY_TOKEN`) instead of `DEVPASS_TOKEN`.
- **`sk-...` (RunGate Gateway Key):** Created in RunGate (`rungate_create_gateway_key`) for LLM inference at `https://gateway.rungate.ru/v1`, with optional ruble spending cap (`limitRub`).
- **`adopto_<tenant>_<project>_<key>_...` (Adopto Project Key):** Fine-grained 12-scope key for backend email/event integration.
- **Public keys (`access_key` in Submito, `siteKey` in Answero):** Safe for client HTML, restricted to public intake/widget endpoints only.

## 4. Setting Up `@devpass/mcp`

1. Direct the user to sign in at `https://id.devpass.ru` and copy their `DEVPASS_TOKEN` (`dp_live_...`).
2. Configure their client:

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

> **Note:** Rekuro has its own dedicated MCP server at `https://api.rekuro.ru/v1/mcp` (authenticated with `REKURO_API_KEY`), separate from `@devpass/mcp`.

## 5. Integrating DevPass SSO (`@devpass/auth-sdk`)

When building a service that delegates auth and plans to `id.devpass.ru`:
- Use `@devpass/auth-sdk` (`createSatelliteAuth`, `verifyDevPassApiKey`, `createEntitlementsResolver`, `createDevpassEventsHandler`).
- Standard routes: `/auth/login`, `/auth/callback` (atomic one-time Redis ticket exchange), `/auth/sign-out`, and `/internal/devpass/events` (Ed25519-signed webhook with `x-devpass-signature` and `x-devpass-timestamp`, 300s skew tolerance).
- Cookie naming follows RFC 6265bis: `__Host-dp_sid` for single-origin prod, `__Secure-dp_sid` when sharing a wildcard `Domain`, `dp_sid` on `localhost`.

## Troubleshooting

- **`DEVPASS_TOKEN is not set` in MCP:** Verify the `env` block in the MCP config JSON and restart the MCP client/server.
- **403 / quota exceeded when creating a resource:** Call `devpass_status` to check active plan limits; either clean up unused resources or advise upgrading to `devpass_pro` / standalone Pro.
