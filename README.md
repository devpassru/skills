# DevPass Agent Skills

Official [Agent Skills](https://agentskills.io) for the **DevPass** developer ecosystem (`devpass.ru`).

Each skill turns your AI coding agent into a **domain expert**—guiding architecture decisions, quota/plan selection, security boundaries, end-to-end integration workflows, and troubleshooting—rather than just listing API or MCP methods.

Compatible with Claude Code, Cursor, Codex, Antigravity, Windsurf, and any agent supporting `npx skills`.

## Installation

Install all DevPass ecosystem skills:

```bash
npx skills add devpassru/skills
```

Or install a specific product skill:

```bash
npx skills add devpassru/skills --skill devpass
npx skills add devpassru/skills --skill submito
npx skills add devpassru/skills --skill rundeploy
npx skills add devpassru/skills --skill rungate
npx skills add devpassru/skills --skill runping
npx skills add devpassru/skills --skill runport
npx skills add devpassru/skills --skill adopto
npx skills add devpassru/skills --skill answero
npx skills add devpassru/skills --skill rekuro
```

## Available Skills

| Skill | Product | Domain Expertise |
| :--- | :--- | :--- |
| [`devpass`](skills/devpass/SKILL.md) | **DevPass** | Ecosystem architecture, plan/quota strategy (`Free` vs `devpass_pro`), token hygiene, `@devpass/mcp` & `@devpass/auth-sdk` SSO |
| [`submito`](skills/submito/SKILL.md) | **Submito** | Form intake patterns (HTML `303` vs SPA `Accept: application/json`), `botcheck` & ALTCHA anti-spam, payload limits, error diagnosis |
| [`rundeploy`](skills/rundeploy/SKILL.md) | **RunDeploy** | Static/SPA/TMA & Bun serverless (`/api/*`) hosting, packaging rules, `@devpass/rundeploy` CLI, `rdtok_...` CI tokens, custom SSL & rollbacks |
| [`rungate`](skills/rungate/SKILL.md) | **RunGate** | Choosing AI Gateway (`gateway.rungate.ru/v1`) vs Smart DNS (DoH + Zero-TLS-Decryption SNI proxy), model selection, OS/router setup & diagnostics |
| [`runping`](skills/runping/SKILL.md) | **RunPing** | SRE check selection (`http`, `keyword`, `json`, `tcp`, auto SSL), anti-flap tuning, SSRF boundaries, `/s/:slug` status pages & incident triage |
| [`runport`](skills/runport/SKILL.md) | **RunPort** | Zero-install SSH tunnels (`srv.runport.ru:2222`), webhook debugging, `--review` UI annotation loop, Vite `allowedHosts` & HMR fixes |
| [`adopto`](skills/adopto/SKILL.md) | **Adopto** | Lifecycle email architecture, monotonic `profileVersion`, consent/unsubscribe rules, idempotent events, safe template/flow authoring & DNS checks |
| [`answero`](skills/answero/SKILL.md) | **Answero** | Support routing (`ai_first`, `hybrid`, `telegram_only`), KB & `systemPrompt` design, Shadow DOM embed, Telegram Reply + Adopto fallback |
| [`rekuro`](skills/rekuro/SKILL.md) | **Rekuro** | T-Bank payment orchestration, integer kopecks & `Idempotency-Key`, subscriptions/trials/Smart Dunning, Outbox webhooks & `/v1/mcp` debugging |

## Using with DevPass MCP

Most management actions (creating forms, deploying archives, adding monitors, starting tunnels) work seamlessly when [`@devpass/mcp`](https://github.com/devpassru/mcp) is connected with your `DEVPASS_TOKEN` from [id.devpass.ru](https://id.devpass.ru). Rekuro exposes its own dedicated MCP server at `https://api.rekuro.ru/v1/mcp`.

## License

MIT
