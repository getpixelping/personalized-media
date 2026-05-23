# Klaviyo's WhatsApp dynamic header image mechanism

**Question**: Does Klaviyo expose Meta's per-recipient dynamic header
image capability for WhatsApp templated messages? If yes, how exactly?

**Answer**: YES, but only via the *indirect* "URL stored in property"
pattern — not raw token substitution inside a URL string.

## Mechanism

Klaviyo's WhatsApp template **does support dynamic header images**, but
not by exposing Meta's `components.parameters[0].image.link` field
directly. Instead the image header points at a Klaviyo variable
(profile property, event property, or catalog tag) that *resolves to a
URL at send time*.

**What works on Klaviyo's surface:**
- ✅ Static image header (fixed image baked in at template approval)
- ✅ Dynamic image header *referencing* a Klaviyo variable that resolves
  to a complete URL — e.g. `{{ event.image_url }}` mapped to a property
  containing `https://render.example.com/img/abc.png`
- ❌ A URL string with embedded `{{ person.first_name }}` tokens that
  Klaviyo substitutes inside the URL — no documentation confirms this
  works for the header image field, only for body text

**The send pattern:**
1. Pre-compute the personalized URL (per recipient)
2. POST to Klaviyo's Events API with the URL as a property:
   ```json
   {
     "metric": "Cart Abandoned with Personalized Image",
     "profile": { "$email": "..." },
     "properties": {
       "image_url": "https://render.example.com/img/abc-recipient-hash.png",
       "first_name": "Sarah",
       "product_name": "AirPods",
       "price": "$129"
     }
   }
   ```
3. Merchant's Klaviyo flow listens for that metric, triggers a WhatsApp
   send. The Klaviyo Template's header image is bound to
   `{{ event.image_url }}`.
4. Klaviyo fetches our PNG and passes the URL through to Meta as the
   header media.

## Constraints

- WhatsApp template header: `.jpg`, `.jpeg`, `.png`, max **5 MB**
  (matches Meta)
- No animated GIF in image headers (image type is static only)
- SMS/MMS has a separate **600 KB** cap; WhatsApp does **not**
- Templates submitted **from inside Klaviyo to Meta** for approval —
  no "import already-approved Meta template into Klaviyo" flow

## API access

- **No public REST endpoint** equivalent to Meta's `messages` with
  `image.link` per send for WhatsApp templates
- The Templates API is **email-only**
- The new **Conversations API** supports SMS and WhatsApp but is for
  free-form session messages (not approved template sends with header
  media), and requires OAuth + feature enablement
- The **Campaigns API revision 2026-04-15** added WhatsApp as a channel
  in Beta — full schema not surfaced; worth verifying scope before
  locking architecture
- **Supported programmatic send path**: Events API → triggered flow

## Implication for our architecture

This finding forced [ADR-0004](../adr/0004-events-api-orchestration.md) —
we always orchestrate via Events API, never via simpler URL-paste.

## Sources

- [How to create a WhatsApp template](https://help.klaviyo.com/hc/en-us/articles/40116644987675)
- [How to send transactional WhatsApp messages](https://help.klaviyo.com/hc/en-us/articles/46625983798299)
- [Dynamic image in SMS/MMS pattern](https://help.klaviyo.com/hc/en-us/articles/1260806102230)
- [Create Event API](https://developers.klaviyo.com/en/reference/create_event)
- [Templates API overview (email-only)](https://developers.klaviyo.com/en/reference/templates_api_overview)
- [Campaigns API overview (rev 2026-04-15, WhatsApp beta)](https://developers.klaviyo.com/en/reference/campaigns_omni_api_overview)
