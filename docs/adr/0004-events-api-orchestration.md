# Events API orchestration, not URL-paste-only mode

Our integration with Klaviyo uses a single send pattern: we POST a
custom metric + properties (including the personalized PNG URL) to
Klaviyo's Events API on a Shopify webhook trigger; the merchant's
Klaviyo flow listens for that metric and sends the WhatsApp/RCS
message with our URL referenced as a Klaviyo template variable.

The simpler "merchant pastes our URL template into Klaviyo's flow
directly" mode was discarded after research confirmed Klaviyo does
**not** substitute `{{ person.first_name }}` tokens *inside* a URL
string for the header image field — only as a complete URL stored in a
property. See [research/04-klaviyo-whatsapp-image-mechanism.md](../research/04-klaviyo-whatsapp-image-mechanism.md).

## Consequences

- We are always the orchestrator; we always know when a personalized
  image is being requested
- We need to scaffold a Klaviyo flow per merchant during onboarding
  (auto-create via Klaviyo's Flows API, then merchant submits the
  template for Meta/Google approval through Klaviyo's UI)
- No "send WhatsApp now with this header URL" public REST endpoint
  exists on Klaviyo — sends are always mediated by Klaviyo's flow
  engine. If Klaviyo's 2026-04 Campaigns API beta exposes direct
  send-with-media at GA, that could simplify orchestration in v1.5
- Reliability story is two-leg: Shopify webhook → our backend, then
  our backend → Klaviyo Events API. Both need idempotency and retries.
