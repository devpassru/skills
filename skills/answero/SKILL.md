---
name: answero
description: "Embed the Answero AI + Telegram support chat widget (<script src='https://answero.ru/widget.js' data-answero='...' defer>) into HTML, React, or Astro sites, and manage FAQ knowledge bases or conversations via Answero API/MCP. Use when adding customer support chat or AI FAQ bots to a website."
license: MIT
---

# Answero AI + Telegram Support Widget

## Overview

Answero (`https://answero.ru`) provides an embeddable AI + Telegram support chat widget (`ai_first`, `hybrid`, or `telegram_only` routing), an FAQ knowledge base, and operator conversation management.

## When to Use

- Embedding the Answero chat widget on a landing page, SaaS dashboard, or docs site.
- Creating a site (`siteKey`), adding FAQ/knowledge documents, or replying to support conversations via `@devpass/mcp`.

## Widget Embed

Paste the `<script>` tag before `</body>` (or in your root layout in Astro/React/Next.js) using the public `siteKey`:

```html
<script src="https://answero.ru/widget.js" data-answero="YOUR_SITE_KEY" defer></script>
```

- `data-answero` uses the public `siteKey` (safe for browser HTML).
- Restrict `allowedDomains` in the site settings so only your production domains can load the widget.

## MCP Workflow (`@devpass/mcp`)

1. `answero_list_sites` — check existing sites and plan limits.
2. `answero_create_site` — create a site (`name`, optional `allowedDomains`, `mode`: `ai_first | hybrid | telegram_only`, `themeColor` `#RRGGBB`, `greetingMessage`, `position`: `bottom_right | bottom_left`, `aiEnabled`, `aiModel`, `systemPrompt`).
3. `answero_get_widget_snippet` — fetch the ready-to-paste `<script>` snippet for a `site_id`.
4. `answero_add_knowledge` / `answero_list_knowledge` — populate FAQ or raw text docs for AI first-line replies.
5. `answero_list_conversations` / `answero_get_conversation` / `answero_reply_conversation` / `answero_update_conversation_status` — inspect and handle live support threads.

## Common Pitfalls

- **Domain mismatch:** If the widget fails to initialize on production, verify the domain is listed in `allowedDomains` (or leave `allowedDomains` empty during initial setup).
- **Exposing `DEVPASS_TOKEN`:** Only `siteKey` goes into `<script data-answero="...">`; management APIs (`/api/v1/sites`) require `Authorization: Bearer $DEVPASS_TOKEN` server-side or via MCP.
