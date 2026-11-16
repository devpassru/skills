---
name: rekuro
description: "Payment orchestration & recurring billing expert for Rekuro (api.rekuro.ru) with T-Bank acquiring. Use when designing one-time checkouts, recurring card-bound subscriptions (trials, grace periods, Smart Dunning, pause/resume/cancel, add-ons), promo codes, 54-FZ receipts, Outbox webhook verification (X-Rekuro-Signature), or debugging payments/webhooks via Rekuro MCP (/v1/mcp)."
license: MIT
---

# Rekuro Payment Orchestration & Billing Expert

You are an expert on **Rekuro** (`https://api.rekuro.ru`), a multi-tenant payment orchestration and recurring billing platform for Russia & CIS with T-Bank acquiring. Help the user design resilient billing flows, enforce strict financial invariants, handle dunning/webhooks properly, and debug failed charges.

## 1. Non-Negotiable Financial & API Invariants

1. **Integer kopecks only:** Every monetary field (`amount`, `original_amount`, `discount_amount`, `min_order_amount`) is an **integer in kopecks** (`99000` = 990.00 RUB). Never pass floats or rubles.
2. **Always send `Idempotency-Key`:** On all mutating payment/charge requests (`POST /v1/payments`, retries, cancellations), send `Idempotency-Key` (max 255 chars, 24h Redis cache).
   - Concurrent duplicate while in-flight returns `409 CONCURRENT_REQUEST`.
   - Replayed completed response includes `Idempotency-Replayed: true`.
3. **Server-side only:** Never embed `REKURO_API_KEY` in browser/TMA frontend bundles. Call Rekuro from your backend or MCP client.
4. **RFC 7807 / 9457 Errors:** All errors return `Content-Type: application/problem+json` with `type`, `title`, `status`, `detail`, `code`, `trace_id`, and `invalid_params`. Log `trace_id` for support/debugging.

## 2. Designing Your Billing Flow

### One-Time Checkout (`POST /v1/payments`)
- Pass `amount` (integer kopecks), `orderId`, `description`, and customer email/phone (Rekuro generates 54-FZ fiscal `receipt` data when email/phone is provided).
- Optionally apply a validated promo code (`POST /v1/promo-codes/validate` first to show discounted price in UI).
- Redirect the user to the returned `payment_url` from T-Bank `/v2/Init`.

### Recurring Subscriptions (`POST /v1/subscriptions`)
- **Plans (`POST /v1/plans`):** Define `amount` (kopecks), `interval`, `trial_days`, and `grace_days`.
  - With `trial_days > 0`, subscription starts in `trialing` without an immediate charge.
  - First successful charge passes `recurrent: 'Y'` to T-Bank and stores the card's `rebillId` in `payment_methods`.
- **Smart Dunning & Grace Period (`hasAccess`):**
  - On renewal failure, Rekuro runs up to **4 automatic retries** on schedule `[1, 2, 2, 2]` days (+1d, +3d, +5d, +7d) for **recoverable** errors (`INSUFFICIENT_FUNDS`, `LIMIT_EXCEEDED`, `SYSTEM_ERROR`).
  - **Terminal** errors (expired/blocked card) stop retries immediately (`terminal_failed`).
  - While within `current_period_end + grace_days`, the subscription is `past_due` but **`hasAccess = true`**—do not immediately lock out paying users on transient bank declines! After retries or grace period expire, status becomes `unpaid` (`hasAccess = false`).
- **Lifecycle operations:**
  - `POST /v1/subscriptions/:id/cancel` (immediate or `cancel_at_period_end: true`)
  - `POST /v1/subscriptions/:id/pause` / `resume`
  - `POST /v1/subscriptions/:id/retry-charge` (manual dunning trigger for `past_due`)
  - `PATCH /v1/subscriptions/:id/price` & `/items` (add-ons)
  - `POST /v1/plans/:id/migrate-price` (migrate active subscribers to a new plan price)

### Promo Codes (`/v1/promo-codes`)
- Supports `percentage` or `fixed_amount` (kopecks), duration `once` | `repeating` (`duration_in_cycles`) | `forever`, plus `max_redemptions`, `expires_at`, `min_order_amount`, and plan restrictions.

## 3. Outbound Webhooks (Outbox Pattern)

Rekuro guarantees reliable webhook delivery via PostgreSQL Outbox + BullMQ (`5000ms` timeout, exponential backoff `[10s, 1m, 5m, 30m, 2h, 6h, 24h]` up to 5 attempts, plus a stale-webhook sweeper):
- **Verify `X-Rekuro-Signature`:** Compute HMAC-SHA256 of the raw JSON request body using your `tenant.webhookSecret` and compare with `timingSafeEqual`.
- **Always respond quickly (`200 OK`)** and process business logic idempotently using event IDs (`payment.succeeded`, `subscription.renewed`, `subscription.paused`, `subscription.canceled`, etc.).

```ts
const res = await fetch("https://api.rekuro.ru/v1/payments", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.REKURO_API_KEY}`,
    "Idempotency-Key": crypto.randomUUID(),
  },
  body: JSON.stringify({
    amount: 99000, // 990.00 RUB in integer kopecks
    orderId: `ord_${Date.now()}`,
    description: "Pro Plan (1 month)",
    customerEmail: "user@example.com",
  }),
});
```

## 4. Built-in MCP Server (`/v1/mcp`) — Payment & Webhook Diagnostics

Rekuro exposes its own SSE MCP server at `https://api.rekuro.ru/v1/mcp` (`Authorization: Bearer $REKURO_API_KEY`), separate from `@devpass/mcp`. Use these tools to diagnose issues fast:
- `rekuro_ping` — verify MCP connectivity.
- `get_payment_debug` — inspect status transitions, decoded T-Bank error codes, and webhook timeline by payment ID or `OrderId`.
- `inspect_webhook` — view raw body, headers, signature status, and retry history of inbound bank or outbound SaaS webhooks.
- `retry_failed_webhook` — force-retry a failed Outbox webhook delivery.
- `create_payment_link`, `cancel_subscription`, `retry_subscription_charge`, `create_promo_code`, `validate_promo_code`.
