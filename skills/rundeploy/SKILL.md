---
name: rundeploy
description: "Expert deployment engineer for RunDeploy (rundeploy.ru). Use when deploying static sites, SPAs (React/Vite/Astro), Telegram Mini Apps, or Bun serverless APIs (/api/*), configuring @devpass/rundeploy CLI or CI/CD deploy tokens (rdtok_...), setting up custom domains with automatic SSL, managing AES-256-GCM env variables, or rolling back releases."
license: MIT
---

# RunDeploy Hosting & Serverless Expert

You are an expert on **RunDeploy** (`https://rundeploy.ru`). Help the user choose the right project architecture, package and deploy cleanly (via CLI, MCP, or CI/CD), wire up custom domains and encrypted secrets, and debug serverless or release issues.

## 1. Choose the Right Architecture & Limits

| Workload | Supported? | How to Configure |
| :--- | :--- | :--- |
| **Static site (HTML/CSS/JS, Astro)** | Yes (Angie NVMe) | `spaFallback: false` (returns 404 on unknown paths) |
| **SPA (React, Vite, Vue)** | Yes (Angie NVMe) | `spaFallback: true` (default; `try_files $uri $uri/ /index.html =404`) |
| **Telegram Mini App (TMA)** | Yes | Deploy as SPA (`spaFallback: true`); use `https://<slug>.rundeploy.ru` or custom domain in `@BotFather` |
| **Serverless API (`/api/*`)** | Yes (Pro & Business only) | Place `.ts`/`.js`/`.mts`/`.mjs` handlers in `api/` adjacent to `dist/` |
| **Docker / Micro-containers** | **Not yet implemented** | Schema reserves `'container'`, but runtime only executes static/SPA + Bun serverless functions |

### Key Plan Limits
- **Free:** 1 project, **50 MB** unpacked bundle, 10 GB traffic, 3 retained releases, **no custom domains**, **0 serverless requests** (403 on `/api/*`).
- **Pro (290 RUB / `devpass_pro`):** 5 projects, **200 MB** bundle, 50 GB traffic, 10 releases, custom domains + auto SSL, **50,000 serverless req/mo** (10 concurrent).
- **Business (590 RUB):** 30 projects, **1 GB** bundle, 300 GB traffic, 100 releases, custom domains + auto SSL, **500,000 serverless req/mo** (50 concurrent).

## 2. Serverless Functions (`/api/*`) Contract

RunDeploy automatically detects an `api/` directory and bundles handlers with `Bun.build` (`target: "bun"`, `format: "esm"`).

- **File-based routing (priority: static > dynamic > catch-all):**
  - `api/index.ts` -> `/api`
  - `api/hello.ts` -> `/api/hello`
  - `api/users/[id].ts` -> `/api/users/:id` (params passed in `x-rundeploy-params` header)
  - `api/proxy/[...slug].ts` -> `/api/proxy/*`
- **Handler signature:**
  ```ts
  export default async function handler(
    req: Request,
    env: Record<string, string>
  ): Promise<Response | unknown> {
    return Response.json({ ok: true, keySet: Boolean(env.API_SECRET) });
  }
  ```
- **Runtime limits:** **10s execution timeout** (`504 Gateway Timeout` via `SIGKILL`), **5 MB max request body**, isolated child process (`bun run --smol`) with host secrets stripped—only your project's `env_variables` are injected into `env`.

## 3. Packaging Rules (Prevent Common Rejections)

Before uploading a `.zip`, `.tar.gz`, or `.tgz` archive (or using the CLI):
1. **Root must contain `index.html` or `api/`:** Archive the *contents* of `dist/` (`cd dist && zip -r ../release.zip .` or `tar -czf release.tar.gz -C dist .`), **not** the `dist/` folder itself, otherwise you get `missing_index_html`.
2. **Never include `node_modules/` or `.git/`:** Max **5,000 files** and unpacked size must fit `max_bundle_mb` (streaming Zip Bomb check).
3. **No symlinks or `..`:** Symlinks (`0o120000` / tar `'1'`/`'2'`) and path traversal are rejected immediately with `symlink_forbidden`.

## 4. Deployment Methods

### Option A: Official CLI (`@devpass/rundeploy`) — Fastest for Humans & CI
```bash
# One-off or global install
npx @devpass/rundeploy login            # saves token to ~/.config/devpass/credentials.json
npx @devpass/rundeploy ./dist -n my-app # auto-detects dist/build/out/public and adjacent api/
```
Useful CLI commands:
- `rundeploy env set KEY VALUE` / `rundeploy env ls` / `rundeploy env rm KEY`
- `rundeploy domains add app.example.com` -> `rundeploy domains verify app.example.com`
- `rundeploy logs --limit 50` (inspect `/api/*` logs, 7-day retention, last 100 entries)
- `rundeploy rollback [dep_id]` (atomic symlink switch; zero 404 window)

### Option B: CI/CD with Scoped Deploy Token
Never put `DEVPASS_TOKEN` in CI secrets if you only need to deploy one project. Create a scoped deploy token (`rdtok_...`) via `rundeploy_create_deploy_token` and pass it as `RUNDEPLOY_TOKEN` (or `Authorization: Bearer rdtok_...` to `POST /api/v1/projects/:id/deployments`).

### Option C: `@devpass/mcp` Workflow
1. `rundeploy_list_projects` -> `rundeploy_create_project` (`slug` must match `^[a-z0-9](?:[a-z0-9-]{1,46}[a-z0-9])?$`).
2. `rundeploy_set_env` (keys must match `^[A-Z0-9_]{1,64}$`; values are encrypted with **AES-256-GCM** and never returned by `list_env`).
3. `rundeploy_deploy_archive` (`project_id`, `archive_path`, optional `commit_sha`, `commit_message`).
4. Every release also gets an immutable preview URL: `https://<depId>--<slug>.rundeploy.ru`.

## 5. Custom Domains & Automatic SSL

1. Requires **Pro** or **Business** (`custom_domains_pro_required` 403 on Free).
2. Add domain via `rundeploy domains add` or `rundeploy_add_domain`.
3. Configure DNS:
   - Subdomain (`app.example.com`): **CNAME** -> `cname.rundeploy.ru`.
   - Apex (`example.com`): **A** record -> server IP shown by `add_domain`.
4. Trigger verification (`rundeploy_verify_domain`). Angie automatically issues a Let's Encrypt certificate via HTTP-01 (`pending` -> `issuing` -> `active`) and sets up a **301 canonical redirect** from `<slug>.rundeploy.ru` to your custom domain.

## Troubleshooting

- **404 on page refresh in React/Vue:** Project was created with `spaFallback: false`. Update via `PATCH /api/v1/projects/:id`.
- **Env var not visible in frontend `import.meta.env`:** RunDeploy `env_variables` are **runtime secrets for `/api/*` serverless functions**. Frontend build-time vars (`VITE_*`, `PUBLIC_*`) must be injected before `npm run build`.
- **Cannot rollback to old release:** Releases beyond plan retention (3/10/100) are `pruned`; failed/uploading releases cannot be rolled back to.
