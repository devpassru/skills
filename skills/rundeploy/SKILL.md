---
name: rundeploy
description: "Build, package, and deploy static sites, SPAs, and Telegram Mini Apps to RunDeploy (rundeploy.ru), manage atomic rollbacks, custom domains, and encrypted environment variables. Use when deploying a frontend build, configuring a *.rundeploy.ru subdomain, or managing RunDeploy releases via MCP or REST API."
license: MIT
---

# RunDeploy Hosting & Releases

## Overview

RunDeploy (`https://rundeploy.ru`) hosts static sites, SPAs (React/Vite/Astro), and Telegram Mini Apps with atomic symlink releases, `*.rundeploy.ru` subdomains, automatic SSL for custom domains, and AES-256-GCM encrypted environment variables.

## When to Use

- Deploying a built frontend (`dist/`, `build/`, `out/`) as a `.zip` or `.tar.gz` archive to RunDeploy.
- Creating a new RunDeploy project, attaching a custom domain, setting env variables, or rolling back a release.

## Deployment Workflow

1. **Build locally first:** Run the project's build command (e.g., `npm run build` or `bun run build`) and verify the output directory contains `index.html` at the archive root.
2. **Package cleanly:** Archive the *contents* of the build directory (not the parent folder, and never `node_modules` or `.git`):
   ```bash
   cd dist && zip -r ../release.zip .
   ```
   Or with `tar.gz`:
   ```bash
   tar -czf release.tar.gz -C dist .
   ```
3. **Deploy atomically:** Upload via `rundeploy_deploy_archive` (MCP) or `POST /api/v1/projects/:projectId/deployments` (`multipart/form-data` with field `file`).

## MCP Tools (`@devpass/mcp`)

- `rundeploy_list_projects` — list projects, production URLs, active deployments, and domains.
- `rundeploy_create_project` — create a project (`name`, `slug` matching `^[a-z0-9](?:[a-z0-9-]{1,46}[a-z0-9])?$`).
- `rundeploy_deploy_archive` — upload a `.zip` or `.tar.gz` archive (`project_id`, `archive_path`, optional `commit_sha`, `commit_message`).
- `rundeploy_list_deployments` / `rundeploy_rollback_deployment` — inspect history and instantly roll back to a previous release.
- `rundeploy_add_domain` / `rundeploy_verify_domain` / `rundeploy_list_domains` — attach custom domains and trigger SSL issuance.
- `rundeploy_set_env` / `rundeploy_list_env` — manage encrypted environment variables.

## Security & Archive Rules

- **No symlinks or path traversal:** Archives containing `..`, absolute paths, or symlinks escaping the release directory are rejected.
- **Bundle size limit:** Respect the plan's `max_bundle_mb` limit; exclude source maps or unused assets if the archive is too large.
- **Root `index.html`:** Ensure `index.html` is at the top level of the archive (`./index.html`), not nested inside `./dist/index.html`.
