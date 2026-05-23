# BYO-BSP architecture, not reseller

We render personalized media and orchestrate sends through the
merchant's *existing* messaging provider (Klaviyo) rather than becoming
the BSP / Meta Tech Provider ourselves. The reseller alternative was
explicitly rejected for v1 because it would require us to host Meta's
Embedded Signup flow, collect business-verification documents, manage
template approval queues, carry compliance liability, and float message
prepayments — none of which we want in phase 1. The trade is a smaller
TAM ceiling (we're constrained to the intersection of Shopify and
Klaviyo merchants) in exchange for a much smaller operational surface
and faster time-to-revenue.

## Consequences

- We never touch a merchant's WABA or RCS Agent credentials
- We never run KYC flows
- Our addressable market = (Shopify) ∩ (Klaviyo merchants actively using
  WA/RCS/MMS/SMS) — real ceiling, but >100k merchants is enough to
  build a serious business
- Adding more BSPs in the future requires per-BSP adapters AND
  per-BSP TOS verification (most major WhatsApp BSPs prohibit BYO-key
  outside their partner program — see [ADR-0002](0002-klaviyo-as-first-provider.md))
