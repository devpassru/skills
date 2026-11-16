---
name: submito
description: "Expert guide for Submito form backend (app.submito.ru). Use when adding contact, waitlist, lead, or feedback forms to HTML, Astro, React, or Next.js sites without custom backend code, configuring ALTCHA captcha, honeypot spam protection, redirects, idempotency, webhooks, or debugging POST /v1/submit errors."
license: MIT
---

# Submito Form Backend Expert

You are an expert on **Submito** (`https://app.submito.ru`), an agent-friendly form backend. Help the user choose the right submission pattern, wire up spam protection and notifications cleanly, and avoid real platform pitfalls.

## 1. Choose the Right Integration Pattern

Before writing form code, decide which submission flow fits the user's stack:

| Use Case | Pattern | Response Handling |
| :--- | :--- | :--- |
| **Static HTML / Astro (no JS needed)** | Standard `<form action="https://app.submito.ru/v1/submit" method="POST">` | Server returns `303 See Other` to `/submit/result?receipt=<id>` or a whitelisted `redirect` URL |
| **React / Next.js / Vite SPA** | `fetch("https://app.submito.ru/v1/submit", { method: "POST", body: formData, headers: { Accept: "application/json" } })` | Server returns JSON (`201 Created` or `200 OK` on replay) without page reload |
| **Backend-to-backend / programmatic** | `POST /v1/submit` with `Content-Type: application/json` + `Idempotency-Key` | Flat JSON only; ideal with `Idempotency-Key` (24h TTL) to prevent duplicate leads |

## 2. Hard Platform Constraints (Know Before Designing)

Verify the user's requirements against actual Submito limits so you never promise unsupported behavior:

1. **No file uploads:** `POST /v1/submit` accepts **text fields only** (`application/json`, `application/x-www-form-urlencoded`, or `multipart/form-data` with text parts). Any part with `filename="..."` or binary content-type fails with `400 invalid_multipart`. If the user needs file attachments, advise uploading to object storage first and submitting the resulting URL as a text field.
2. **Flat payload & size limits:**
   - Max body size: **64 KB** (`65,536` bytes), UTF-8 only (`Content-Encoding` compression is rejected).
   - Max **100 fields** per submission; max **8 KB** per field value; max **20 items** for array fields.
   - Field names must match `/^[a-zA-Z][a-zA-Z0-9_]{0,63}$/` (no `__proto__`, `constructor`, `prototype`, no nested JSON objects, no duplicate keys).
3. **Single public endpoint:** All submissions go to `https://app.submito.ru/v1/submit` with `access_key` in the body/hidden input. Never invent per-form URLs.
4. **Telegram notifications:** Delivery in the backend currently supports **email** (verified owner email) and **signed webhooks** (HMAC-SHA256, `pg-boss` outbox with 4 retries). If the user wants Telegram alerts right now, configure a **webhook** pointing to their Telegram relay/bot endpoint rather than relying on native Telegram delivery.
5. **Plan quotas:** Free = 3 active forms, 100 submissions/mo (30d retention); Pro = 10 forms, 2,000/mo (365d); Business = 50 forms, 20,000/mo (365d).

## 3. Spam Protection & UX Best Practices

### Honeypot (`botcheck`)
Submito has a built-in reserved honeypot field named `botcheck`. If present and non-empty, Submito rejects the submission with `400 invalid_request` (`honeypot_filled`).
Add an off-screen/hidden field that real users don't fill:
```html
<input type="checkbox" name="botcheck" style="display:none" tabindex="-1" autocomplete="off" />
```

### ALTCHA Proof-of-Work Captcha (`captcha_mode: "altcha"`)
For high-traffic public forms, enable `altcha` mode instead of annoying visual CAPTCHAs:
- Client loads `/assets/captcha/3.5.0/submito.js` (and `altcha.js`) or fetches a signed 5-minute PoW challenge via `POST /v1/captcha/challenge` with `{ "access_key": "..." }`.
- Submit the solved token in the `captcha_token` field (max 8,192 bytes).

### Custom Redirects (HTML Forms)
To redirect after an HTML `<form>` submit, include `<input type="hidden" name="redirect" value="https://yoursite.com/thanks" />` **and** ensure `https://yoursite.com` is registered in the form's `redirect_origins` (HTTPS required in production; `localhost`/`127.0.0.1` allowed in dev).

## 4. Production-Ready Examples

### HTML / Astro (with honeypot)
```html
<form action="https://app.submito.ru/v1/submit" method="POST">
  <input type="hidden" name="access_key" value="YOUR_ACCESS_KEY" />
  <input type="checkbox" name="botcheck" style="display:none" tabindex="-1" autocomplete="off" />
  <input type="email" name="email" required placeholder="your@email.com" />
  <textarea name="message" placeholder="Your message"></textarea>
  <button type="submit">Send</button>
</form>
```

### React / SPA (with `Accept: application/json`, error handling & idempotency)
```tsx
import { useState } from "react";

export function ContactForm({ accessKey }: { accessKey: string }) {
  const [status, setStatus] = useState<"idle" | "sending" | "success" | "error">("idle");

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setStatus("sending");

    const formData = new FormData(e.currentTarget);
    formData.set("access_key", accessKey);

    try {
      const res = await fetch("https://app.submito.ru/v1/submit", {
        method: "POST",
        body: formData,
        headers: {
          Accept: "application/json",
          "Idempotency-Key": crypto.randomUUID(),
        },
      });
      setStatus(res.ok ? "success" : "error");
    } catch {
      setStatus("error");
    }
  }

  if (status === "success") return <p>Thank you! Your request was received.</p>;

  return (
    <form onSubmit={handleSubmit}>
      <input type="checkbox" name="botcheck" style={{ display: "none" }} tabIndex={-1} autoComplete="off" />
      <input type="email" name="email" required placeholder="name@company.com" />
      <textarea name="message" placeholder="How can we help?" />
      <button type="submit" disabled={status === "sending"}>
        {status === "sending" ? "Sending..." : "Submit"}
      </button>
      {status === "error" && <p>Failed to send. Please try again.</p>}
    </form>
  );
}
```

## 5. Provisioning & Troubleshooting

- **MCP Workflow (`@devpass/mcp`):** `submito_list_forms` -> `submito_create_form` (returns `form_id` and one-time `access_key`) -> `submito_get_snippet`.
- **Key rotation:** `access_key` is shown only once at creation (stored as SHA-256). If lost or leaked, rotate via `POST /v1/forms/:id/rotate-key`.
- **Error Diagnosis:**
  - `400 invalid_request`: Check `invalid_access_key`, `honeypot_filled`, nested JSON objects, or invalid field names (must start with a letter).
  - `409 idempotency_conflict`: Same `Idempotency-Key` reused with a different payload.
  - `413 payload_too_large`: Body > 64 KB or single field > 8 KB.
  - `429 quota_exceeded` / `rate_limited`: Monthly quota hit or burst limit exceeded (check `Retry-After`).
  - **HTML redirect instead of JSON in SPA:** Missing `Accept: application/json` header on `fetch()`.
