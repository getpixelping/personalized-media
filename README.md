# Personalized Media

A Shopify app that renders per-recipient personalized images (and later GIFs)
for use as header media in the merchant's existing **Klaviyo** WhatsApp,
RCS, MMS, and SMS flows. We're a content layer, not a delivery layer.

## Status

**Pre-development.** Plan validated through 15 brainstorming exchanges +
6 targeted research dispatches. Implementation has not started.

## Where to find things

| Topic | File |
|---|---|
| Current plan + roadmap | [docs/plan.md](docs/plan.md) |
| Architecture decisions | [docs/adr/](docs/adr/) |
| Validation research | [docs/research/](docs/research/) |
| Domain language + glossary | [CONTEXT.md](CONTEXT.md) |

## The shape, in one paragraph

We're a **Shopify App Store** app. Merchant installs us, pastes their
Klaviyo private API key, picks an abandoned-cart template from our
gallery, and maps Shopify cart data to template variables. On a Shopify
`checkouts/update` webhook (cart abandoned), we render a personalized
PNG and POST a custom event to Klaviyo's Events API with the rendered
image URL. The merchant's Klaviyo flow (which we scaffold during
onboarding) listens for that event and triggers a WhatsApp/RCS/MMS
template send, with our PNG as the header media. Klaviyo handles
delivery and compliance. We never touch the WhatsApp Business Account,
never do KYC, never become a Meta Tech Provider.

## The wedge

**Movable Ink** ships pixel-personalized media for Klaviyo's email and
MMS channels — but explicitly **not for WhatsApp or RCS**. That gap
inside Klaviyo's ~117k Shopify-brand footprint is the v1 territory we're
claiming.

## What this folder is

A developer-support-site preserving the brainstorming and validation
work that produced the current plan, so a future contributor (or
future-self) can reconstruct *why* the architecture looks the way it
does — not just *what* it does.
