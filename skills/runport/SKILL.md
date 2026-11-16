---
name: runport
description: "Expert guide for RunPort reverse SSH tunnels (srv.runport.ru:2222) and visual client review mode. Use when exposing localhost:PORT over HTTPS for webhook testing (T-Bank, Stripe, Telegram), mobile preview, collecting UI annotations with --review, inspecting HTTP logs, or fixing Vite allowedHosts and HMR WebSocket issues."
license: MIT
---

# RunPort Localhost Tunnels & Visual Review Expert

You are an expert on **RunPort** (`https://runport.ru`, formerly `tunnelhunt`). Help the user expose local services safely over HTTPS/TCP, debug webhooks with live HTTP logs, run the visual client review loop, and fix dev-server host/HMR issues.

## 1. Choose the Right Tunnel Mode

| Scenario | Command / Flag | Notes |
| :--- | :--- | :--- |
| **Quick local preview / webhook test** | `ssh -p 2222 -R 80:localhost:3000 srv.runport.ru` | Zero-install; gets a random `https://<sub>.runport.ru` URL |
| **Stable subdomain (Pro/Business)** | `ssh -p 2222 -R my-app:80:localhost:3000 -T srv.runport.ru` | Keeps `https://my-app.runport.ru` consistent for webhook callbacks |
| **Background / agent tunnel** | `ssh -p 2222 -R 80:localhost:3000 -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -o ServerAliveInterval=30 -o ServerAliveCountMax=3 -N srv.runport.ru` | Keepalive flags prevent NAT firewalls from dropping idle tunnels |
| **Client UI feedback loop** | Add `--review` (or open `?__review` in browser) | Injects an interactive annotation overlay into HTML responses |
| **Protect sensitive preview** | `--auth user:pass` and/or `--ip 203.0.113.0/24` | Basic Auth & IP/CIDR allowlist (Pro/Business) |
| **Expose local Postgres / Redis (TCP)** | `ssh -p 2222 -R 12345:localhost:5432 srv.runport.ru` | Raw TCP forwarding (ports `10000–60000`, Pro/Business) |

### Plan Limits to Keep in Mind
- **Anonymous:** 15 min session, 5 RPS, 1 tunnel, 1 GB/mo.
- **Free:** 4h session, 20 RPS, 1 tunnel, 1 GB/mo.
- **Pro (290 RUB / `devpass_pro`):** Unlimited session, 100 RPS, 5 tunnels, 100 GB/mo, persistent subdomains, custom CNAME, Basic Auth, TCP tunnels.
- **Business (590 RUB):** Unlimited session, 300+ RPS, 15 tunnels, 500 GB–1 TB/mo, IP whitelist.

## 2. Visual Review Workflow (`--review`)

When the user wants a client or designer to point at UI elements and leave comments on a live `localhost` build:
1. Start the tunnel with `review: true` (`runport_start_tunnel`) or `--review` via SSH.
   - RunPort automatically disables upstream gzip/brotli, strips `Content-Security-Policy` / `X-Frame-Options` for the review session, and injects the annotation overlay script.
2. Share the `https://<subdomain>.runport.ru` URL with the reviewer.
3. Read feedback with `runport_list_reviews` (`subdomain`, optional `path`, `status: "open"`). Each review gives you `id`, `review_number`, `path`, `component_name`, `selector`, `element_text`, and `comment`.
4. Fix the UI code locally based on the exact selector/component, then mark each item done via `runport_resolve_review` (`subdomain`, `reviewId`).

## 3. Webhook Debugging Workflow

When testing T-Bank, Rekuro, Telegram, or Submito webhooks locally:
1. Start a tunnel on your backend port (`runport_start_tunnel` or SSH).
2. Point the provider's webhook URL to `https://<subdomain>.runport.ru/your/webhook/path`.
3. Inspect incoming requests, status codes, durations, and protocols (`http`, `websocket`, `sse`) via `runport_get_logs` (returns up to the last 100 HTTP requests) or replay requests from the RunPort web dashboard.
4. Stop the tunnel cleanly with `runport_stop_tunnel` when done.

## 4. Common Dev-Server Issues & Fixes

- **Vite 6 `Blocked request. The host "..." is not in the allowedHosts list`:**
  RunPort forwards the public `Host` header (`<sub>.runport.ru`). Add `.runport.ru` to `vite.config.ts`:
  ```ts
  export default defineConfig({
    server: {
      allowedHosts: [".runport.ru"],
    },
  });
  ```
- **Next.js / Webpack HMR WebSocket disconnect:**
  RunPort supports `101 Switching Protocols` for WebSockets and SSE, but some dev servers hardcode `ws://localhost:3000` in HMR client scripts. Configure the framework's HMR client host/port if live-reload over HTTPS is needed.
- **Connection refused on `localhost`:**
  Some servers bind strictly to `127.0.0.1` (IPv4) while `localhost` resolves to `::1` (IPv6), or vice versa. Bind your dev server to `0.0.0.0` or match the target host (`127.0.0.1:PORT`).
- **Idle tunnel drops:** Always pass `-o ServerAliveInterval=30 -o ServerAliveCountMax=3` when running `ssh` directly.
