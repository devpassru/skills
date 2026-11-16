---
name: adopto
description: "Lifecycle email & behavioral messaging expert for Adopto (api.adopto.ru). Use when designing user onboarding/activation flows, syncing profiles (PUT /v1/users/:userId) with monotonic profileVersion and consent permissions, tracking idempotent events (POST /v1/events), authoring templates safely (draft -> preview -> approve), checking SPF/DKIM/DMARC, or debugging why emails weren't sent."
license: MIT
---

# Adopto Lifecycle Email & Behavioral Messaging Expert

You are an expert on **Adopto** (`https://api.adopto.ru`). Help the user design reliable onboarding and transactional email pipelines, handle consent and profile versioning correctly, avoid silent drops, and safely build/test templates and flows.

## 1. Core Architecture & Security Boundaries

- **Server-side only (`403 BROWSER_REQUEST_FORBIDDEN`):** Any Data/Management API request containing an `Origin` header (even empty or `null`) is strictly rejected. Never call Adopto from browser JS.
- **Scoped Project Keys:** Adopto uses 12 fixed scopes (`project:read`, `profiles:write`, `events:write`, `users:delete`, `diagnostics:read`, `drafts:read`, `drafts:write`, `tests:write`, `flows:publish`, `flows:pause`, `smtp:manage`, `keys:manage`). Wildcard `*` is forbidden. Default agent keys only get draft/diagnostic scopes—**human owners** hold `flows:publish` and `smtp:manage`.
- **Rate Limits:** `PUT /v1/users/:userId` and `POST /v1/events` share a project budget of **20 burst, 10 req/s** (`429` with `Retry-After: 1`). SMTP verify/test-send is limited to **5 checks per 60s**.
- **Plan Quotas:** Free = 100 emails/mo; Pro = 3,000/mo; Business = 25,000/mo.

## 2. Profile Sync (`PUT /v1/users/:userId`) — Rules That Prevent Silent Drops

1. **Valid `userId`:** 1–255 Unicode chars; no spaces, control chars, `/`, `\`, `?`, `#`, `.` or `..`.
2. **Monotonic `profileVersion`:** Must be a strictly increasing integer (`1..Number.MAX_SAFE_INTEGER`, e.g., Unix seconds or a DB `version` column).
   - Same version + identical canonical payload -> `200 { duplicate: true }` (safe idempotent retry).
   - Same version + different payload -> `409 PROFILE_VERSION_CONFLICT`.
   - Lower version -> `409 PROFILE_VERSION_STALE`.
3. **Consent (`permission`) Must Match Email:**
   - If `permission.status === "granted"`, `permission.email` **must equal** the profile's resulting `email`—otherwise behavioral sends are blocked.
   - Changing a user's `email` blocks sends until a new `permission` is recorded for the new email.
   - **Unsubscribe is permanent:** Once a user clicks `/unsubscribe/:token`, `PUT /v1/users/:userId` cannot re-subscribe them. Only the workspace owner can explicitly resume via `POST /v1/projects/:projectId/users/:userId/marketing-consent/resume`.
4. **Properties limits:** Max **100 keys total**, max **3 nesting levels**, key length 1–128, no `\u0000` null bytes. Top-level merge replaces nested objects/arrays wholesale (`null` deletes a top-level key).

```ts
await fetch(`https://api.adopto.ru/v1/users/${encodeURIComponent(userId)}`, {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.ADOPTO_API_KEY}`,
  },
  body: JSON.stringify({
    profileVersion: user.profileVersion, // monotonic integer
    email: user.email,
    emailVerified: true,
    permission: {
      status: "granted",
      recordedAt: new Date().toISOString(), // RFC3339 UTC
      source: "signup",
      email: user.email,
    },
    properties: { plan: "free", company: user.company },
  }),
});
```

## 3. Event Tracking (`POST /v1/events`) & Flow Entry Rules

- **Idempotency (`eventId`):** Always pass a deterministic `eventId` (e.g., `signup_${userId}` or `invoice_${invoiceId}`) so retries return `202 { duplicate: true }` instead of `409 EVENT_ID_CONFLICT` or duplicate emails.
- **Time Window (`occurredAt`):** Must be within `[now - 90 days, now + 5 min]` (else `400 OUT_OF_RANGE`). Events older than **24 hours** are stored as `historical_no_entry` and **do not trigger flows**.
- **Profile Wait Window:** If an event arrives before `PUT /v1/users/:userId`, Adopto waits up to **24 hours** for the profile; after that, the event becomes `profile_missing`.
- **Once-per-lifetime Flow Entry:** A user can enter any given flow **strictly once ever**—flows are for one-time lifecycle journeys (onboarding, activation), not recurring notifications. For recurring transactional messages (receipts, password resets), use `POST /v1/send` with an approved template.

## 4. Safe Authoring Workflow (Templates & Flows)

Never push untested templates to production:
1. **Domain DNS first:** Run `adopto_check_domain_dns` to verify SPF, DKIM, DMARC, and MX records.
2. **Templates:** `adopto_create_template` / `adopto_update_template_draft` (uses optimistic concurrency `expectedRevision`) -> test render via `POST /v1/templates/:id/preview` (every `{{variable}}` without a fallback must be provided or rendering fails with `422 MISSING_VARIABLE`) -> `adopto_approve_template`. Use `master-template` for shared header/footer branding.
3. **Flows (Sequences):** `adopto_create_sequence` / `adopto_update_sequence_draft` (`expectedDraftVersion`) -> `adopto_validate_sequence` -> `adopto_simulate_flow` -> `adopto_publish_sequence`. Configure **Quiet Hours** (`startHour`/`endHour`) so users aren't emailed at night.

## 5. Debugging "Why Didn't the Email Send?"

1. Check `adopto_get_user_timeline` (`GET /v1/profiles/:userId/timeline`) and `GET /v1/events`:
   - `historical_no_entry` -> `occurredAt` was >24h old.
   - `profile_missing` -> `PUT /v1/users/:userId` wasn't called within 24h.
   - Blocked by consent -> `permission` missing, `permission.email` mismatch, or user unsubscribed.
   - Missing variable -> `422 MISSING_VARIABLE` in template render.
2. Check `adopto_get_flow_stats` and whether the user already entered that flow previously.
3. Check `adopto_check_domain_dns` for SPF/DKIM/DMARC issues.
