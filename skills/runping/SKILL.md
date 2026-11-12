---
name: runping
description: "Set up RunPing uptime, keyword, JSON, TCP, and SSL certificate monitors and public status pages (/s/:slug). Use when adding availability monitoring for a deployed site or API, checking downtime incidents, or publishing a status page via RunPing API or MCP."
license: MIT
---

# RunPing Uptime, SSL & Status Pages

## Overview

RunPing (`https://runping.ru`) monitors HTTP(S), Keyword, JSON, and TCP endpoints, tracks SSL certificate expiration, and hosts public status pages at `/s/:slug`.

## When to Use

- Creating uptime or SSL expiration monitors for a newly deployed site or API.
- Checking current monitor status, latency, or recent downtime incidents.
- Publishing a public status page exposing selected monitors.

## MCP Tools (`@devpass/mcp`)

- `runping_list_monitors` — list monitors (optional filters: `status` in `pending|up|degraded|down|paused`, `search`).
- `runping_create_monitor` — create an HTTP(S), Keyword, JSON, or TCP monitor.
- `runping_get_incidents` — fetch recent downtime incidents and failure reasons.
- `runping_pause_monitor` / `runping_resume_monitor` — pause or resume checks by monitor ID.
- `runping_create_status_page` — create a public status page (`/s/:slug`) for selected monitors.

## REST API (`https://runping.ru`)

Authenticate requests with `Authorization: Bearer $DEVPASS_TOKEN`:
- `GET /api/v1/monitors` — list monitors
- `POST /api/v1/monitors` — create monitor

## Security & Target Rules

- **Public targets only (SSRF protection):** RunPing blocks private/internal addresses (`localhost`, `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `::1`, `fc00::/7`, `fe80::/10`), including after DNS resolution and across HTTP redirects.
- **Public status pages:** Never rely on monitor headers or auth tokens being visible on public status pages—they are stripped from `/api/public/status-pages/:slug`.
- **Testing local services:** If you need to monitor or preview a local port, expose it publicly first with RunPort (`https://<subdomain>.runport.ru`).
