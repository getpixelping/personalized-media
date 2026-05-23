# Meta's 10-minute URL cache — implications for our architecture

**Question**: Does Meta's image URL caching break per-recipient
personalization?

**Answer**: No — actually slightly beneficial to our architecture.

## How Meta's cache works

Meta's WhatsApp Cloud API internally caches each image URL **for a
static 10-minute window** after the first fetch. From Meta's
[Send Messages guide](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages/):

> "WhatsApp Cloud API internally caches the asset for a static time
> period of 10 minutes... If you don't want the API to reuse the
> cached asset within the 10 minute time period, you can append a
> random query string to the asset link, and the API will treat it as
> a new asset, fetch it from your server, and cache it for 10 minutes."

The cache key is the **full URL string** including query parameters.

## Why it's not a blocker for our design

Our URL pattern is `https://render.example.com/img/{template}/{variable_hash}.png`.
Two recipients with different personalization data produce different
hashes → different URLs → different Meta cache entries. Two recipients
with identical content share one URL → cache hit → cost-efficient.

The flow naturally aligns:

1. Shopify webhook fires at T=0 (cart abandoned)
2. We render the personalized PNG → store in R2 at T+50ms
3. We POST Klaviyo event with image URL at T+100ms
4. Klaviyo flow triggers WhatsApp send at T+1hr (recovery delay)
5. Klaviyo calls Meta API → Meta fetches our URL → first cache entry
6. Subsequent recipients with the same URL hit Meta's cache for 10 min

Our pre-rendered PNG sits in R2 by the time Meta fetches. Meta's fetch
latency = CDN latency, not render latency.

## Constraints to design around

1. **URLs must be stable for at least 10 minutes after first fetch.**
   Don't delete or change content served at a URL within that window.
   Our `(template_id, variable_hash)` cache key gives stability
   automatically.

2. **No signed URLs with short expirations.** If we used
   `?signature=...&expires=300s`, Meta's cache fetches could find an
   expired signature. Use public-bucket access with hard-to-guess
   hash-based paths instead — security via unguessability, not via
   expiring signatures.

3. **Template version belongs in the URL path**, not as a cache-bust
   parameter. If we change a template design, increment the version:
   `https://render.example.com/img/v2/{template_id}/{variable_hash}.png`.
   Old URLs keep serving the old image during their cache lifetime;
   new sends get the new design.

4. **Batch sends are fine.** A single URL might cache-miss and re-fetch
   a couple of times across a long batch, but each re-fetch hits our
   R2 CDN (already populated), not our renderer. Bandwidth cost only.

## What needs verification

The 10-minute number applies to Meta-direct Cloud API. Under
Klaviyo-mediated path, **Klaviyo's API hands the URL to Meta** —
Meta's caching behavior should still apply, but if Klaviyo also caches
or proxies image URLs at their layer with different TTLs, that's
additional behavior to verify. Worth confirming in the same spike that
validates Klaviyo's Campaigns API beta scope.

## The real reliability concern (not caching)

The bigger risk isn't Meta's caching — it's our R2/Worker availability
during Meta's fetch window. If our image hosting goes down between
webhook fire (T=0) and Meta fetch (T+1hr), Meta gets a 5xx and either
drops the media or fails the send.

**Mitigations**:
- Pre-render aggressively at webhook receipt
- Monitor R2/Worker uptime
- Configure Klaviyo to send without header media if the URL fails
  (graceful degradation — recipient still gets the message body)

## Sources

- [Meta WhatsApp Cloud API — Send Messages](https://developers.facebook.com/docs/whatsapp/cloud-api/guides/send-messages/)
- [Meta WhatsApp Cloud API — Media reference](https://developers.facebook.com/docs/whatsapp/cloud-api/reference/media/)
