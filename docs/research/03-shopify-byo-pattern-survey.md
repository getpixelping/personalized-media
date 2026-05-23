# Shopify App Store: BYO-via-API-key architectural pattern survey

**Question**: Is the "BYO via API key paste, enhance existing tool"
integration pattern commercially proven on the Shopify App Store?

**Method**: Surveyed 10 representative Shopify apps that follow this
pattern, capturing install/review counts and pricing tiers.

## Findings

### Pattern is heavily proven — for Klaviyo specifically

| App | Reviews | Rating | Pricing | What they do |
|---|---|---|---|---|
| Judge.me | 39,401 | 5.0 | Free / $15 | Reviews → push to Klaviyo |
| Loox | 7,902 | 4.9 | Free / $49.99 / $299.99 | Visual reviews → Klaviyo |
| Smile.io | 4,232 | 4.9 | Free / $15 / $79 / $199 | Loyalty → Klaviyo flows |
| Yotpo Reviews | 4,417 | 4.8 | Free / $15 / $119 | Reviews → Klaviyo |
| Stamped | 3,627 | 4.7 | $23 / $99 / $199 / $299+ | Reviews + loyalty → Klaviyo |
| Okendo | 1,344 | 4.8 | Free / $19 / $119 / $299 | Reviews + Loyalty → Klaviyo/Postscript/Gorgias |
| LoyaltyLion | 514 | 4.6 | Free / $199 / Custom | Loyalty → Klaviyo + Attentive |
| Triple Whale | 89 | 4.0 | Free / $149 / $219 | Multi-source analytics |
| Polar Analytics | 110 | 4.8 | $750 base (GMV-based) | Multi-source analytics |

**Pricing pattern for BYO-enhancement apps on Shopify**:
- Entry self-serve: **$15–$50/mo**
- At-scale: **$99–$299/mo**
- Premium analytics tier: $149–$750/mo

### The critical negative finding

**BYO-WhatsApp-BSP is NOT an established pattern on the Shopify App
Store.** Zero apps found that ask merchants to "paste your existing
Wati / Interakt / AiSensy API key." Every WhatsApp app on Shopify
(Wati, Interakt, AiSensy, Zoko, BiteSpeed) **IS** the BSP — the
merchant signs up for WhatsApp Business API through the app, and the
app bills directly for messages plus a SaaS fee.

This gap is explained by the BSP TOS audit
([research/02](02-bsp-tos-audit.md)) — most WhatsApp BSPs prohibit the
BYO pattern in their TOS. It's a structural cause, not an oversight.

### Multi-BSP adapter pattern

**Zero Shopify apps** run a "pick which messaging provider to send
through at runtime" architecture for WhatsApp/RCS. LoyaltyLion, Okendo,
Stamped support multiple email/SMS providers as parallel-single
integrations but never as runtime adapters.

This makes multi-adapter architecture for messaging a *double-novel*
shape on Shopify. v1 stays with single-Klaviyo to avoid both novelty
risks; multi-adapter is deferred indefinitely.

## Implication for our architecture

- BYO-Klaviyo is on extremely well-trodden ground commercially
- Pricing assumption: enter at $19–49/mo, scale to $99–199/mo (matches the survey)
- We don't need to invent a new UX motif — Shopify merchants already understand "paste your Klaviyo key"
