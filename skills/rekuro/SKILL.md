---
name: rekuro
description: "Integrate Rekuro payment orchestration and recurring billing (https://api.rekuro.ru) with T-Bank acquiring, subscriptions, promo codes, webhooks, and the built-in Rekuro MCP server (/v1/mcp). Use when adding payments, recurring subscriptions, checkout links, promo codes, or payment/webhook debugging via Rekuro."
license: MIT
---

# Rekuro Payment Orchestration & Recurring Billing

## Overview

Rekuro (`https://api.rekuro.ru`) is a multi-tenant payment orchestration and recurring billing platform for Russia & CIS with T-Bank acquiring, subscription scheduling, promo codes, reliable Outbox webhooks, and a built-in MCP server.

## When to Use

- Creating one-time checkout links or recurring card-bound subscriptions via Rekuro REST API or MCP.
- Managing billing plans, customers, subscriptions (`cancel`, `pause`, `resume`, `retry-charge`), and promo codes.
- Debugging failed payments, inspecting inbound/outbound webhook deliveries, or replaying failed Outbox webhooks.

## REST API (`https://api.rekuro.ru`)

- **Base URL:** `https://api.rekuro.ru`
- **Auth header:** `Authorization: Bearer $REKURO_API_KEY` (tenant API key)
- **Amounts:** All monetary amounts are integers in **kopecks** (e.g. `99000` = 990.00 RUB).
- **Errors:** Returned in RFC 7807 Problem Details format (`application/problem+json`).

### Key Endpoints

- **Customers:**
  - `POST /v1/customers` — create a customer
  - `GET /v1/customers/{id}` — get customer details
- **Plans:**
  - `POST /v1/plans` — create a billing plan
  - `GET /v1/plans` — list plans
  - `GET /v1/plans/{id}` — get plan details
  - `PATCH /v1/plans/{id}` — update plan
  - `GET /v1/plans/{id}/price-history` — get plan price history
  - `POST /v1/plans/{id}/migrate-price` — migrate active subscriptions to new price
- **Payments:**
  - `POST /v1/payments` — initialize a payment / checkout session (supports `Idempotency-Key`)
  - `GET /v1/payments` — list payments
  - `GET /v1/payments/{id}` — get payment details
  - `POST /v1/payments/{id}/cancel` — cancel or refund a payment
- **Subscriptions:**
  - `POST /v1/subscriptions` — create a subscription
  - `GET /v1/subscriptions` — list subscriptions
  - `GET /v1/subscriptions/{id}` — get subscription details
  - `POST /v1/subscriptions/{id}/cancel` — cancel subscription
  - `POST /v1/subscriptions/{id}/pause` — pause subscription
  - `POST /v1/subscriptions/{id}/resume` — resume paused subscription
  - `POST /v1/subscriptions/{id}/retry-charge` — manually retry a `past_due` charge
  - `PATCH /v1/subscriptions/{id}/price` — update subscription price
- **Promo Codes:**
  - `POST /v1/promo-codes` — create a promo code
  - `GET /v1/promo-codes` — list promo codes
  - `GET /v1/promo-codes/{id}` — get promo code details
  - `PATCH /v1/promo-codes/{id}` — update promo code
  - `DELETE /v1/promo-codes/{id}` — deactivate/delete promo code
  - `POST /v1/promo-codes/validate` — validate a promo code and calculate discount

### Example: Create Payment Session

```ts
const res = await fetch("https://api.rekuro.ru/v1/payments", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.REKURO_API_KEY}`,
    "Idempotency-Key": crypto.randomUUID(),
  },
  body: JSON.stringify({
    amount: 99000, // 990.00 RUB in kopecks
    orderId: `ord_${Date.now()}`,
    description: "Pro Plan (1 month)",
    customerEmail: "user@example.com",
  }),
});
```

## Built-in MCP Server (`/v1/mcp`)

Rekuro exposes its own MCP server at `https://api.rekuro.ru/v1/mcp` (authenticated via `Authorization: Bearer $REKURO_API_KEY`).

Available MCP tools:
- `rekuro_ping` — check Rekuro MCP server availability.
- `create_payment_link` — create a payment session and return a checkout URL (one-time or plan subscription with card binding).
- `get_payment_debug` — deep diagnostics for a payment by ID or `OrderId` (status transitions, bank error decoding, webhook timeline).
- `inspect_webhook` — inspect inbound bank webhooks or outbound SaaS webhook deliveries (raw body, headers, signature status, retry history).
- `retry_failed_webhook` — force-retry an outbound webhook delivery via Outbox.
- `cancel_subscription` — cancel a subscription with reason, audit log, and `subscription.canceled` Outbox event.
- `create_promo_code` — issue a new promo code (including personal discount codes).
- `validate_promo_code` — validate a promo code for an amount, plan, and customer and compute the discount.
- `retry_subscription_charge` — manually retry charging a `past_due` subscription (Smart Dunning).

## Common Pitfalls

- **Passing rubles instead of kopecks:** Always pass integer kopecks (`99000` for 990 RUB), never floating-point rubles.
- **Missing `Idempotency-Key`:** Always send an `Idempotency-Key` header on `POST /v1/payments` and charge retries to prevent duplicate charges.
- **Exposing tenant API keys in frontend code:** Never embed `REKURO_API_KEY` in browser bundles; call Rekuro only from backend services or MCP clients.
