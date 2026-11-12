---
name: submito
description: "Integrate Submito form backend (POST /v1/submit with access_key) into HTML, React, or Astro websites, or provision forms via Submito MCP tools. Use when adding contact, waitlist, feedback, or lead forms without writing custom backend code."
license: MIT
---

# Submito Form Backend Integration

## Overview

Submito (`https://app.submito.ru`) is an agent-friendly form backend. All public form submissions go through a single endpoint (`POST /v1/submit`) authenticated by a public `access_key`.

## When to Use

- Adding a contact, waitlist, lead-capture, or feedback form to an HTML, Astro, React, Next.js, or Vite site.
- Provisioning a new form endpoint or inspecting existing forms via `@devpass/mcp`.

## Integration Rules

1. **Single public endpoint:** Always submit forms to `https://app.submito.ru/v1/submit` (or `SUBMITO_API_URL` if overridden). Do not invent per-form endpoint URLs.
2. **Public `access_key`:** Pass `access_key` in a hidden `<input type="hidden" name="access_key" value="..." />` or inside `FormData` / JSON body.
3. **Scope separation:** `access_key` only grants permission to submit to that form. Never expose `DEVPASS_TOKEN` (`dp_live_...`) or management tokens in frontend code.

## Code Examples

### HTML / Astro

```html
<form action="https://app.submito.ru/v1/submit" method="POST">
  <input type="hidden" name="access_key" value="YOUR_ACCESS_KEY" />
  <input type="email" name="email" required placeholder="your@email.com" />
  <textarea name="message" placeholder="Your message"></textarea>
  <button type="submit">Send</button>
</form>
```

### React (Fetch + `FormData`)

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
        headers: { Accept: "application/json" },
      });
      setStatus(res.ok ? "success" : "error");
    } catch {
      setStatus("error");
    }
  }

  if (status === "success") {
    return <p>Thank you! Your request was received.</p>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" name="email" required placeholder="name@company.com" />
      <button type="submit" disabled={status === "sending"}>
        {status === "sending" ? "Sending..." : "Submit"}
      </button>
      {status === "error" && <p>Failed to send. Please try again.</p>}
    </form>
  );
}
```

## MCP Workflow (`@devpass/mcp`)

When `@devpass/mcp` is connected:
1. Call `submito_list_forms` to check existing forms.
2. Call `submito_create_form` (with `name`, optional `notify_email`, `telegram_chat_id`, `redirect_url`) to receive `form_id` and `access_key`.
3. Optionally call `submito_get_snippet` with `access_key` and `framework` (`html` | `react` | `astro`).

## Common Pitfalls

- **Missing `Accept: application/json` in SPA requests:** Without `Accept: application/json`, `fetch()` may follow an HTML redirect instead of returning JSON status.
- **Confusing `DEVPASS_TOKEN` with `access_key`:** Management APIs (`/v1/forms`) require `Authorization: Bearer $DEVPASS_TOKEN` server-side; browser forms must only use `access_key`.
