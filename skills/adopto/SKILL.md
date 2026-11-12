---
name: adopto
description: "Integrate Adopto user lifecycle synchronization (PUT /v1/users/:userId) and behavioral event tracking (POST /v1/events) at https://api.adopto.ru to trigger onboarding email sequences. Use when adding user signup/activation tracking or managing Adopto sequences."
license: MIT
---

# Adopto Behavioral Tracking & Onboarding Sequences

## Overview

Adopto (`https://api.adopto.ru`) tracks user profiles and behavioral lifecycle events to trigger or complete automated onboarding email sequences.

## When to Use

- Syncing user profiles (`PUT /v1/users/:userId`) on registration or plan changes.
- Sending idempotent lifecycle events (`POST /v1/events`) such as `user.registered`, `subscription.paid`, or custom activation events.
- Inspecting configured onboarding sequences via `@devpass/mcp`.

## API Integration

- **Base URL:** `https://api.adopto.ru`
- **Auth header:** `Authorization: Bearer $ADOPTO_API_KEY` (or `$DEVPASS_TOKEN`)

### 1. Sync User Profile (`PUT /v1/users/:userId`)

Send a monotonic `profileVersion` (e.g. Unix timestamp in seconds) so out-of-order updates never overwrite newer state:

```ts
await fetch(`https://api.adopto.ru/v1/users/${encodeURIComponent(userId)}`, {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.DEVPASS_TOKEN}`,
  },
  body: JSON.stringify({
    profileVersion: Math.floor(Date.now() / 1000),
    email: user.email,
    emailVerified: true,
    permission: {
      status: "granted",
      recordedAt: new Date().toISOString(),
      source: "app",
      email: user.email,
    },
    properties: { plan: "free" },
  }),
});
```

### 2. Track Behavioral Event (`POST /v1/events`)

Always include a deterministic or unique `eventId` for idempotency and an ISO-8601 UTC `occurredAt`:

```ts
await fetch("https://api.adopto.ru/v1/events", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.DEVPASS_TOKEN}`,
  },
  body: JSON.stringify({
    eventId: crypto.randomUUID(),
    userId,
    name: "user.registered",
    occurredAt: new Date().toISOString(),
    properties: { source: "web" },
  }),
});
```

## MCP Tools (`@devpass/mcp`)

- `adopto_sync_user` — sync a user profile (`user_id`, `email`, optional `profile_version`, `properties`).
- `adopto_track_event` — send an event (`user_id` or `user_email`, `event_name`, optional `event_id`, `occurred_at`, `properties`).
- `adopto_list_sequences` — list configured email sequences.

## Common Pitfalls

- **Calling Adopto from browser JS:** Never expose `ADOPTO_API_KEY` or `DEVPASS_TOKEN` in client-side code; send profile syncs and events from your backend.
- **Missing `eventId`:** Reuse a stable `eventId` when retrying failed webhook/job deliveries so Adopto deduplicates retries.
