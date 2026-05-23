# Klaviyo public API with merchant key, not Tech Partner Agreement

For v1 we do NOT sign Klaviyo's Technology Partner Agreement. Instead
the merchant pastes their own Klaviyo private API key into our Shopify
app, and we call Klaviyo's REST API authenticated as them. Klaviyo's
public API Terms explicitly permit this pattern — third-party
integrations using customer keys are a documented use case
("treat keys like a password — only share with parties you trust").

## Considered Options

- **Apply for Klaviyo Tech Partner Program upfront** — would give us
  OAuth (cleaner UX), Klaviyo marketplace listing, and co-marketing,
  but costs 4–6 weeks application review + the §10(k) assignment
  restriction in the Tech Partner Agreement (any change of control,
  including the founder incorporating, requires Klaviyo's prior written
  consent).
- **Selected: public API with merchant's BYO key** — zero contractual
  relationship with Klaviyo. No application timeline. No assignment
  friction when the founder incorporates. Trade-off is API-key paste
  UX (slightly worse than OAuth) and no Klaviyo marketplace presence.

## Consequences

- v1 launch timeline shortens by 4–6 weeks (no Klaviyo gating)
- The founder can ship as an individual without signing the Tech
  Partner Agreement — incorporation later doesn't trigger a Klaviyo
  consent step
- Tech Partner status remains an optional v1.5/v2 enhancement if/when
  we want OAuth, marketplace distribution, or co-marketing
- Shopify App Store remains the primary distribution channel — Klaviyo
  marketplace was always a secondary discovery surface
