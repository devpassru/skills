# DevPass Agent Skills

Official [Agent Skills](https://agentskills.io) for the **DevPass** developer ecosystem (`devpass.ru`).

Compatible with Claude Code, Cursor, Codex, Antigravity, Windsurf, and any agent supporting `npx skills`.

## Installation

Install all DevPass ecosystem skills:

```bash
npx skills add devpassru/skills
```

Or install a specific product skill:

```bash
npx skills add devpassru/skills --skill submito
npx skills add devpassru/skills --skill rundeploy
npx skills add devpassru/skills --skill rungate
npx skills add devpassru/skills --skill runping
npx skills add devpassru/skills --skill runport
npx skills add devpassru/skills --skill adopto
npx skills add devpassru/skills --skill answero
npx skills add devpassru/skills --skill devpass
```

## Available Skills

| Skill | Product | What it teaches your agent |
| :--- | :--- | :--- |
| [`devpass`](skills/devpass/SKILL.md) | **DevPass** | Connect `@devpass/mcp`, configure `DEVPASS_TOKEN` (`dp_live_...`), and check account quotas |
| [`submito`](skills/submito/SKILL.md) | **Submito** | Integrate `POST /v1/submit` forms in HTML, React, and Astro using `access_key` safely |
| [`rundeploy`](skills/rundeploy/SKILL.md) | **RunDeploy** | Package builds (`.zip`/`.tar.gz`), deploy atomically, manage custom domains & encrypted env vars |
| [`rungate`](skills/rungate/SKILL.md) | **RunGate** | Configure OpenAI-compatible AI Gateway (`gateway.rungate.ru/v1`) and Smart DNS (DoH) |
| [`runping`](skills/runping/SKILL.md) | **RunPing** | Set up HTTP(S)/Keyword/JSON/TCP/SSL monitors and public status pages (`/s/:slug`) |
| [`runport`](skills/runport/SKILL.md) | **RunPort** | Expose `localhost` over HTTPS via reverse SSH tunnels and visual client review mode |
| [`adopto`](skills/adopto/SKILL.md) | **Adopto** | Sync user profiles (`PUT /v1/users/:userId`) and track lifecycle events (`POST /v1/events`) |
| [`answero`](skills/answero/SKILL.md) | **Answero** | Embed the AI + Telegram support chat widget and manage FAQ knowledge bases |

## Using with DevPass MCP

Most management actions (creating forms, deploying archives, adding monitors, starting tunnels) work seamlessly when [`@devpass/mcp`](https://github.com/devpassru/mcp) is connected with your `DEVPASS_TOKEN` from [id.devpass.ru](https://id.devpass.ru).

## License

MIT
