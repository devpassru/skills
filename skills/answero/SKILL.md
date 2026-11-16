---
name: answero
description: "Customer support & AI chat widget expert for Answero (answero.ru). Use when embedding the Shadow DOM support widget (<script data-answero=...>), choosing between ai_first, hybrid, and telegram_only modes, structuring FAQ knowledge bases (up to 40 active docs), writing escalation systemPrompts, wiring Telegram Reply + Adopto email fallback, or fixing allowedDomains/CORS issues."
license: MIT
---

# Answero AI + Telegram Support Widget Expert

You are an expert on **Answero** (`https://answero.ru`). Help the user choose the right support routing mode, structure an effective knowledge base and AI prompt, embed the widget safely, and wire up Telegram + email fallback.

## 1. Choose the Right Support Mode (`mode`)

| Mode | Behavior | Best For |
| :--- | :--- | :--- |
| **`ai_first`** (default) | AI answers from the Knowledge Base via RunGate (`gpt-4o-mini` default). If the question is outside the KB, involves billing/account issues, or the user asks for a human, AI outputs `[ESCALATE]`, switching status to `waiting_operator` and notifying Telegram. | SaaS products, docs sites, and indie founders wanting 24/7 L1 deflection |
| **`hybrid`** | Same AI first-line behavior, **plus** instant keyword escalation (e.g., `"человек"`, `"оператор"`, `"живой"`, `"позови"`, `"специалист"`) bypasses AI and goes straight to `waiting_operator` in Telegram. | Sales/conversion pages where impatient leads should reach a human immediately |
| **`telegram_only`** | AI is disabled (`shouldRunAi = false`). Every visitor message immediately routes to `waiting_operator` in Telegram. | High-touch consulting or teams that don't want AI replies |

## 2. Knowledge Base & Prompt Design

Answero injects **up to 40 active knowledge documents** (`limit(40)`) and the **last 12 conversation messages** into the AI context:
- **Keep docs focused & scannable:** Each document (`type`: `"faq" | "article" | "raw_text"`) supports `title` (1–200 chars) and `content` (up to 50,000 chars), but shorter, topic-split FAQ entries (pricing, refunds, setup, limits) yield much sharper answers than one giant dump.
- **Plan limits:** Free = 1 site, 20 KB docs, 300 AI replies/mo (with Answero branding); Pro (290 RUB / `devpass_pro`) = 3 sites, 100 KB docs, 2,000 AI replies/mo, no branding; Business = 25 sites (10 in some quotas), 500 KB docs, 25,000 AI replies/mo.
- **Writing `systemPrompt`:**
  - Define tone, language (e.g., Russian/English), and strict boundaries ("Only answer using the provided Knowledge Base; never invent prices or features; escalate billing/refund requests").
  - Toggle `active: false` on outdated docs via `answero_list_knowledge` rather than deleting them permanently.

## 3. Widget Embedding & Architecture

Paste the public `siteKey` snippet before `</body>` (or in your root Astro/React/Next.js layout):
```html
<script src="https://answero.ru/widget.js" data-answero="YOUR_SITE_KEY" defer></script>
```

### Technical Behavior to Know
- **Shadow DOM isolation:** Renders inside `#answero-widget-root` with `attachShadow({ mode: "open" })`—your site's Tailwind/CSS won't break the widget, and widget styles won't leak out.
- **No global JS API:** `widget.js` is self-contained (IIFE); there is **no** `window.Answero` programmatic API.
- **Session & Real-time:** Stores `visitorToken` in `localStorage` (`answero_vt_<siteKey>`) and streams live operator/bot replies over SSE (`/api/v1/widget/conversations/:convId/stream`) via Redis Pub/Sub.
- **Customization:** Configure `themeColor` (`#RRGGBB`), `position` (`bottom_right` | `bottom_left`), `greetingMessage`, and `placeholderText` via `answero_create_site` or the dashboard.

## 4. Telegram Bridge & Email Fallback

- **Telegram two-way Reply:** Link the workspace in the Answero dashboard via the bot deep link (`t.me/<bot>?start=<linkToken>`). When a thread escalates, operators simply **Reply** to the Telegram message card—Answero updates status to `operator_active` and pushes the reply in real time to the visitor's browser via SSE.
- **Email fallback (Adopto):** If the visitor leaves their email (`visitorEmail`) and closes the tab before an operator replies, Answero automatically sends an email notification via Adopto when the operator responds.

## 5. Security & Troubleshooting

- **Token separation:** Only the public `siteKey` belongs in `data-answero="..."`. Never put `DEVPASS_TOKEN` in frontend HTML/JS.
- **Widget fails to load in production:** Check `allowedDomains` on the site (`answero_list_sites`). If `allowedDomains` is non-empty, ensure your exact production hostname (and `localhost` if testing) is included.
- **MCP Workflow (`@devpass/mcp`):** `answero_list_sites` -> `answero_create_site` -> `answero_get_widget_snippet` -> `answero_add_knowledge` -> `answero_list_conversations` / `answero_get_conversation` / `answero_reply_conversation` / `answero_update_conversation_status`.
