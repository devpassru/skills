---
name: runport
description: "Expose a local development server (localhost:PORT) over HTTPS using RunPort reverse SSH tunnels (srv.runport.ru:2222) with optional visual client review mode and HTTP log inspection. Use when testing webhooks locally, sharing a local preview URL, or collecting client UI annotations."
license: MIT
---

# RunPort Localhost Tunnels & Visual Review

## Overview

RunPort (`https://runport.ru`) creates instant reverse SSH tunnels that expose any local port (`localhost:PORT`) on a public `https://<subdomain>.runport.ru` URL without installing a separate binary.

## When to Use

- Exposing a local dev server (`localhost:3000`, `8080`, etc.) to test webhooks (Stripe, Telegram, Submito) or preview on mobile devices.
- Sharing a live local build with a client using visual review mode (`--review`).

## CLI Usage (Zero-Install SSH)

Expose port `3000` on a random subdomain:
```bash
ssh -p 2222 -R 80:localhost:3000 srv.runport.ru
```

Request a custom subdomain and enable visual client review mode:
```bash
ssh -p 2222 -R my-app:80:localhost:3000 -T srv.runport.ru --review
```

## MCP Tools (`@devpass/mcp`)

- `runport_start_tunnel` — start a tunnel for `port` (1–65535), optional `subdomain` (`^[a-z0-9][a-z0-9-]{0,62}$`), and optional `review: true`.
- `runport_get_logs` — inspect recent HTTP request logs passing through the active tunnel.
- `runport_list_reviews` / `runport_resolve_review` — read client visual annotations left in review mode and mark them resolved after fixing the code.
- `runport_stop_tunnel` — close an active tunnel by subdomain.

## Common Pitfalls

- **Local server not listening:** Ensure your dev server is bound and running on `localhost:<port>` before starting the tunnel.
- **Host header / Vite allowedHosts:** If Vite or webpack dev server blocks the `*.runport.ru` Host header, add `.runport.ru` to `server.allowedHosts` in `vite.config.*`.
