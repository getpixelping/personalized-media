# Demand validation matrix — MMS / WhatsApp / RCS × HubSpot / Zoho / Shopify

**Question**: Is there evidence of merchant/marketer demand for
personalized rich-media (image / GIF / video) on each of three
messaging channels (MMS / WhatsApp / RCS) across three platforms
(HubSpot / Zoho / Shopify)?

## Matrix

| | HubSpot | Zoho | Shopify |
|---|---|---|---|
| **MMS** | 🟡 YELLOW. Native + Sakari/Sinch covers per-recipient images via contact-property pattern. Marketplace saturated. 4–6 idea-portal entries, single-digit upvotes. | 🔴 RED. US/Canada only. ~4.2K Ultra Twilio top app. India MMS is essentially non-feature. Zero community demand. | 🔴 **RED. Killer finding**: Postscript shipped Dynamic Product Images in **2019**. Klaviyo has dynamic MMS (≤600 KB). Attentive + **Movable Ink** ship pixel-personalized composites since March 2023. Six-year incumbency. |
| **WhatsApp** | 🟡 YELLOW. ~15 upvotes on send-template idea. HubSpot's docs explicitly say "personalization tokens NOT supported in dynamic URLs" → real gap. But: Meta paused US marketing templates Apr 2025; no idea-portal entries for dynamic image headers specifically. | 🟡→🔴 YELLOW-leaning-RED. Zoho is its own BSP now (lock-in). Marketing Automation 2.0 docs explicitly limit WA header to "one image file." Zero idea-portal entries requesting per-recipient images. | 🟢 YELLOW-GREEN. **Hyperise has NO WhatsApp integration on Shopify** (only Shopify Email). Jan 2026 community thread with 10 replies asking for product images in WA order confirmations. Wati only 4.0★ (lukewarm). |
| **RCS** | 🔴 RED. Zero RCS-specific user ideas on Ideas portal. Discussion is entirely vendor-driven (Sinch and nativeMsg racing for "first RCS in HubSpot"). Supply-push, no demand-pull. | 🔴 RED. No native, no roadmap. Only Indian aggregators (MSG91, Sinch, ValueFirst). Zero community discussion. | 🟡 YELLOW. Klaviyo GA'd RCS **Feb 24, 2026**. **Movable Ink hasn't extended to RCS yet.** RCS traffic +358–550% YoY. Genuinely open lane. But zero merchant threads asking by name → demand is nascent. |

## Headline findings

1. **MMS is closed.** Postscript (2019) + Movable Ink + Klaviyo +
   Attentive own per-recipient dynamic images. Drop MMS from the
   roadmap entirely.

2. **WhatsApp/Shopify wedge survived validation but is softer than
   originally framed.** Hyperise's WhatsApp absence is the most
   defensible finding. But the *flavor* of demand is "insert cart
   product image", not "render the customer's name into pixels" —
   lower differentiation moat than the pixel-personalization thesis
   assumed.

3. **RCS on Shopify is the contrarian opportunity.** Klaviyo GA'd in
   Feb 2026, Movable Ink hasn't extended, channel is in the
   early-adopter window. Risk: merchants aren't asking yet (timing
   problem, not demand problem).

4. **HubSpot demand is louder but tech lanes mostly covered.** SMS/MMS
   via contact-property is well-established. WA marketing templates
   paused in US. RCS has zero user demand.

5. **Zoho is consistently the quietest.** Demand inferred from
   capability gaps, not expressed by users.

## Strategic implication

The path forward is **Klaviyo/Shopify WhatsApp + RCS first**, image
formats. MMS is sunset (saturated). HubSpot/Zoho are separate-product
opportunities, not part of the v1 roadmap.
