# C1–C5: Original Wati-first validation

**Conducted**: early in the brainstorm, when Wati was the proposed first BSP.
**Status**: superseded — see [ADR-0002](../adr/0002-klaviyo-as-first-provider.md).
Retained for context on why Wati was originally attractive before the TOS audit
killed the path.

## C1 — Wati API tier access: GREEN

- `POST /api/v1/sendTemplateMessage` (also `/api/v2/`, Beta) supports
  per-message dynamic media-header URLs via template variables
  (e.g. `{{imageUrl}}`)
- Available on Growth tier (~$49–59/mo); Pro tier ($149/mo) is the
  realistic floor for any merchant doing >330 sends/day (Growth caps at
  10k API calls/mo)
- Bearer-token auth (no OAuth) — merchant pastes their token
- Rate limit: 30/10s on Growth, 60/10s on Pro, 100/10s on Business

Sources: [docs.wati.io](https://docs.wati.io/reference/post_api-v1-sendtemplatemessage),
[support.wati.io](https://support.wati.io/en/articles/11823158-understanding-wati-apis-and-their-usage-limits),
[wati.io/pricing](https://www.wati.io/pricing/)

## C2 — Meta per-send varying image URLs: GREEN

- Cloud API explicitly supports per-recipient image URLs in approved
  templates via `template.components[type=header].parameters[0].image.link`
- Template approval uploads a *sample* image (resumable upload → header_handle);
  the runtime URL is free per send
- Approval turnaround: mostly automated (1–5 min), occasionally 24–48h

Sources: [Meta WhatsApp Cloud API send-templates](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-message-templates/),
[Chatwoot #13159 working payload example](https://github.com/chatwoot/chatwoot/issues/13159)

## C3 — Render feasibility: YELLOW

- Meta-side constraints clear: JPEG/PNG ≤5MB, HTTPS, correct Content-Type
- **Workers Paid tier ($5/mo) required** — Free tier's 10ms CPU kills Satori
- Production path: `workers-og` (Satori + resvg-wasm wrapper)
- **Meta caches by URL for 10 minutes** → personalization MUST live in URL
  (e.g. `?recipientId=hash`), not in body variation only
- Inline fonts as base64; don't fetch Google Fonts at request time
- Animated GIF not allowed in image headers (image type is JPEG/PNG only)
- Meta's image-fetch timeout is undocumented — biggest remaining unknown

Sources: [Meta media reference](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media/),
[Satori on GitHub](https://github.com/vercel/satori),
[Tom Sherman on dynamic OG with Workers](https://tom-sherman.com/blog/dynamic-og-image-cloudflare-workers)

## C4 — Shopify abandoned-cart signal: YELLOW

- Pattern: `checkouts/create` + `checkouts/update` webhooks → Cloudflare
  Queue with `delaySeconds: 3600` → check `completed_at` against
  `orders/create` before firing
- Webhooks at-least-once (8 retries / 4hr backoff), 5s timeout — need
  idempotency on our side
- `checkouts/update` is *chatty* (fires on every meaningful change);
  debouncing logic is real product complexity
- **Protected Customer Data Level 2 review required** for phone access —
  days-to-weeks, no published SLA. Longest pole on App Store launch.
- Phone is only present if the shopper entered contact info — a
  meaningful chunk of abandonments are unaddressable for WhatsApp

Sources: [Shopify webhook retry](https://shopify.dev/changelog/updates-to-webhook-retry-mechanism),
[Protected customer data](https://shopify.dev/docs/apps/launch/protected-customer-data),
[Cloudflare Queues delays](https://developers.cloudflare.com/queues/configuration/batching-retries/)

## C5 — Wati TOS / partnership: YELLOW (superseded by full BSP TOS audit — see [research/02](02-bsp-tos-audit.md))

- Formal Technology Partner Program exists (partners@wati.io)
- TOS §4.1 / §4.6 anti-sublicense / anti-competing-products language
- Precedent strong (Pipedream, Zapier, Zoho Flow integrate via Bearer token)
- **Risk: Wati operates a competing Shopify abandoned-cart app**

Sources: [Wati T&C](https://www.wati.io/terms-and-conditions/),
[Wati Partner Program](https://www.wati.io/become-a-partner/)
