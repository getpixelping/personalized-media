# Personalized Media for WhatsApp + RCS on Shopify — v2 (Klaviyo BYO)

## Context

User's original idea: a plugin for CRM tools (HubSpot, Zoho) that personalizes
images, videos, and GIFs in campaigns — same idea as merge tokens in
email body text, but baked into pixels of the media.

Through ~15 brainstorming exchanges the direction shifted materially. Each
shift was forced by concrete evidence (validation agents, TOS reads,
marketplace surveys), not opinion:

1. **CRM-email → WhatsApp/RCS** — engagement asymmetry (WA 80–95% open
   vs. email 20%) and less crowded competitive lane
2. **HubSpot/Zoho marketplace → Shopify ecosystem** — better cold-start
   distribution + clearer e-commerce use cases
3. **Own-the-stack → BYO-BSP (we render, BSP sends)** — defers KYC
   facilitation, smaller engineering scope
4. **Wati-first → Klaviyo-only** — forced by BSP TOS audit. Wati §4.4
   prohibits credential disclosure to third parties; the BYO-Wati-API-key
   pattern is structurally blocked. Klaviyo's API Terms explicitly
   contemplate exactly this integration pattern.
5. **Single-channel → multi-channel-by-default** — Klaviyo unifies WA +
   RCS + MMS + SMS through one legal integration; no multi-adapter
   complexity needed in v1

## All decisions, cumulative

| # | Decision | Choice |
|---|---|---|
| 1 | Distribution edge | None — cold start, must use platform-native discovery |
| 2 | Channel scope | WhatsApp + RCS first; email out for v1 |
| 3 | Channel priority | WhatsApp via Klaviyo (mature); RCS via Klaviyo (Feb 2026 GA) |
| 4 | Distribution platform | **Shopify App Store** |
| 5 | Send pipeline | **BYO-BSP** — we render, merchant's existing provider sends |
| 6 | First (and only v1) BSP | **Klaviyo** — Wati ruled out by TOS audit |
| 7 | KYC facilitation | **No** in phase 1. Rules out reseller, rules out Meta Direct, rules out Embedded Signup |
| 8 | First use case | Personalized images in **abandoned-cart** recovery (Shopify event → Klaviyo flow) |
| 9 | v1 format | Image only. GIF in v1.1. Video in v2. |
| 10 | Multi-BSP architecture | Adapter pattern *internally*, but Klaviyo is the only v1 adapter |
| 11 | Target merchant | Klaviyo-using Shopify Plus / SMB+ — Western markets, English-speaking |
| 12 | Pricing model assumption | $19–49/mo entry, $99–199/mo at scale (matches BYO-enhancement-app norms) |

## v1 Recommended Approach

A Shopify embedded app that **renders personalized images** and pushes
them into the merchant's existing Klaviyo flows for WhatsApp, RCS, MMS,
and SMS.

The merchant's flow:

1. Install our app from Shopify App Store
2. Connect their Klaviyo account (OAuth via Klaviyo's Tech Partner API)
3. Pick a template from our gallery: canvas with merge-tagged text +
   product image slots — e.g. *"Sarah, your AirPods are still waiting —
   $129"* rendered as a polished WhatsApp/RCS header
4. Map merge fields to Klaviyo profile properties (firstName →
   `$first_name`, productImage → `extra.line_items.0.image`, etc.)
5. We expose a rendered-image URL pattern: `https://us/r/<template>/<recipient_hash>.png`
6. They use this URL inside their existing Klaviyo flow definitions
   (image header in WhatsApp template, RCS card image, MMS media URL)
7. At send time, Klaviyo's existing flow handles delivery; the recipient
   fetches our personalized URL inline; we render the PNG on-the-fly

Our value: the **rendered media URL** + the Shopify-data binding. Klaviyo
keeps everything they're already responsible for (template approval,
delivery, opt-in compliance, billing for messages).

## Why Klaviyo specifically (decision rationale)

Three findings forced this:

1. **TOS audit**: Klaviyo is one of only two major messaging providers
   (along with 360dialog) that *explicitly* permits the BYO-API-key
   pattern. Klaviyo's [API Terms](https://www.klaviyo.com/legal/api-terms)
   anticipate third-party integrations using customer keys; scoped keys
   + OAuth are designed for this. Wati, Interakt, AiSensy, and Sinch all
   restrict or prohibit credential disclosure to third parties.

2. **Architectural-pattern survey**: BYO-via-API-key is *commercially
   proven on Shopify* — but almost exclusively for Klaviyo. Loox (7.9k
   reviews), Judge.me (39k), Yotpo, Okendo, Stamped, Smile.io,
   LoyaltyLion all run this exact model at scale. Pricing is calibrated:
   $15–50/mo entry, $99–299/mo at scale. The "paste your Klaviyo key"
   UX is a no-friction motif Shopify merchants already understand.

3. **Multi-channel through one integration**: Klaviyo's Feb 2026 RCS GA
   plus existing WhatsApp + MMS + SMS support means a single legal
   integration unlocks all our target channels. **Movable Ink** (the
   incumbent for pixel-personalized media) covers Klaviyo's email + MMS
   but explicitly NOT WhatsApp or RCS — that gap inside Klaviyo's
   installed base is our defensible wedge.

## Business Feasibility

### Why this wedge works

- **Engagement asymmetry**: WA/RCS at 80–95% open rates vs. email 20%.
  Personalization ROI is more visible on rich messaging channels.
- **Hyperise has no WhatsApp/RCS integration on Shopify** — the closest
  competitor only does email images. Verified.
- **Movable Ink doesn't extend to WA/RCS via Klaviyo** as of May 2026 —
  verified. Our exact wedge inside Klaviyo's footprint.
- **Klaviyo has ~117k Shopify brands** disclosed. Even if 5–15% activate
  WA/RCS in 2026, that's 6k–18k merchants in our useful TAM.
- **Architectural pattern is proven** — we're not asking merchants to
  trust an unfamiliar integration shape.

### Target buyer

- Shopify Plus + mid-market merchants already on Klaviyo, Western markets
- ACV: $49–99/mo entry, $149–299/mo at scale
- Realistic v1 TAM: 6k–18k merchants (Klaviyo Shopify ∩ active WA/RCS)

### GTM motions

- Shopify App Store (primary discovery + the proven category-leaders'
  acquisition channel)
- Klaviyo Tech Partner Program (co-marketing, listed in their
  integrations directory)
- Content + community: DTC marketing communities, RevOps Slacks, LinkedIn

## Technical Feasibility — Validation Results (cumulative)

### Renderer (still valid from original plan)

- **Render stack**: Cloudflare Workers (Paid tier — $5/mo min, required
  for Satori CPU) + `workers-og` (Satori + resvg-wasm wrapper) + R2 for
  CDN'd PNG storage
- **URL cache key**: `(template_id, variable_hash)` — Meta caches
  per-URL for 10 min so URLs must contain personalization keys
- **Render latency**: target sub-500ms cold, sub-100ms warm; verified
  feasible via community benchmarks
- **Fonts**: inline as base64 in Worker bundle; don't fetch at request
  time
- **Note**: Meta's image-header media constraints (JPEG/PNG ≤5MB, HTTPS,
  correct Content-Type) — same constraints apply when Klaviyo's API
  passes our URL to Meta, since Klaviyo just hands the URL to Meta

### BYO-BSP TOS audit (this turn)

| BSP | BYO-API-key | Notes |
|---|---|---|
| **Klaviyo** | ✅ Allowed | Explicitly contemplated; scoped keys + OAuth designed for it |
| 360dialog | ✅ Allowed (ISV pattern, but canonical flow is Embedded Signup = KYC facilitation, defer) | Full ToS not publicly accessible |
| Twilio | 🟡 Allowed in spirit; canonical pattern is subaccount hierarchy, not BYO key | ISV partner program exists |
| AiSensy | 🟡 TOS silent | Needs sales-rep confirmation |
| Interakt | 🔴 Prohibited by literal text | §5(a) prohibits making Solution available to third party |
| Wati | 🔴 Prohibited absent Technology Partner Program | §4.4 no credential disclosure; §4.1 no transfer |
| Sinch (WhatsApp) | 🔴 Hard block | §2.6 explicitly prohibits disclosing credential to ISV |

### Structural finding

The reason no Shopify app currently does BYO-WhatsApp-BSP is not lack of
imagination — it's that **Meta's Tech Provider model treats the BSP as
the regulated credential holder**, and most BSPs inherit a strict
no-credential-sharing posture downstream. The "canonical" Meta-blessed
ISV pattern is *Tech Provider + Embedded Signup* (= reseller + KYC),
which we explicitly want to defer.

Klaviyo sidesteps this because **Klaviyo is also a registered Tech
Provider** — they hold the Meta relationship; merchants on Klaviyo
already have their WABA registered through Klaviyo; our app uses
Klaviyo's API which Meta has already blessed. We're a tertiary layer on
a legal pyramid, not a credential-sharer.

### Shopify-side (Protected Customer Data still applies)

- **Phone numbers are Level 2 Protected Customer Data** — required for
  WA/RCS/MMS targeting, requires Shopify Partner approval. Days-to-weeks
  review; longest pole on the critical path.
- **Cart abandoned signal**: `checkouts/create` + `checkouts/update` →
  Cloudflare Queue with `delaySeconds: 3600` → check `completed_at`
  before render call
- Webhook delivery is at-least-once, 8 retries / 4hr backoff, 5s timeout
  — need idempotency layer

## v1 Architecture Sketch

```
┌────────────────────┐     ┌─────────────────────┐
│  Shopify Embedded  │     │   Our Backend       │
│  App (React +      │─────│   - Template store  │
│  Polaris)          │     │   - Webhook handler │
│  - Template editor │     │   - Renderer + CDN  │
│  - Klaviyo OAuth   │     │   - Events emitter  │
└────────────────────┘     └─────────────────────┘
         │                          │
         │                          │ POST to Klaviyo Events API
         │                          │  { metric: "Cart Abandoned w/ Image",
         │                          │    properties: { image_url: "https://us/r/abc.png",
         │                          │                  first_name, product, price, ... }}
         │                          ▼
         │                ┌─────────────────────┐
         │                │  Klaviyo Flow Engine│──> WA / RCS / MMS / SMS
         │                │  (listens for our   │    via merchant's existing
         │                │   metric, triggers  │    Klaviyo channels
         │                │   WhatsApp template)│
         │                └─────────────────────┘
         │
         └── Shopify webhooks (cart abandoned, order paid)
```

**One integration pattern — Events-API-orchestrated.** We're always
the orchestrator, never just a URL provider, because Klaviyo's WhatsApp
header image field requires a *complete URL stored in a variable* (it
does NOT support inline token substitution inside a URL string).

The flow:
1. Shopify webhook fires (e.g., cart abandoned)
2. Our backend receives, queries Shopify for full cart + customer context
3. Our renderer generates a personalized PNG, stores at stable URL
   keyed by `(template_id, variable_hash)`
4. Our backend POSTs to Klaviyo's Events API with a custom metric and
   the rendered image URL as a property
5. Klaviyo's flow (defined by merchant during onboarding, or scaffolded
   by us via Klaviyo's Flows API) listens for that metric → triggers
   the WhatsApp/RCS/MMS/SMS template
6. The Klaviyo template's header image field is mapped to
   `{{ event.image_url }}` — Klaviyo fetches our PNG at send time
7. Klaviyo's existing send pipeline handles delivery via merchant's WABA

**Template handling**: Klaviyo submits templates to Meta on the
merchant's behalf — no "import pre-approved Meta template" path exists.
Our app provides Klaviyo-template definitions during onboarding (image
header + body variables), the merchant reviews + submits via Klaviyo UI,
Meta approves through Klaviyo's submission queue.

**No public REST endpoint to "send WhatsApp now with this header URL"
exists.** Klaviyo mediates all sends through their flow/campaign engine.
The 2026-04 Campaigns API beta added WhatsApp as a channel — worth
verifying the beta scope before committing to event-only orchestration
as the sole send mechanism. Could become a v1.5 simplification if the
beta exposes direct send-with-media.

## Key Risks

1. **Klaviyo builds this themselves.** Klaviyo could add pixel-level
   image personalization in 6–12 months and obviate us. Mitigation: move
   fast; the integration plumbing isn't our moat — the renderer quality,
   template gallery, and Shopify-data bindings are.
2. **Shopify Protected Customer Data review** — required for phone
   access; days-to-weeks, no SLA; longest pole on App Store launch
   timeline.
3. **Movable Ink extends to WA/RCS** — possible (probably likely within
   12–18 months given their roadmap velocity). Window to establish our
   wedge is real but finite.
4. **Klaviyo Tech Partner status** — NOT required for v1 (merchant
   pastes their own API key; Klaviyo's API Terms explicitly anticipate
   3rd-party integrations using customer keys). Becomes a v1.5/v2
   enhancement for OAuth + marketplace distribution + co-marketing.
5. **TAM ceiling** — under BYO, our market = (Shopify ∩ Klaviyo). Real
   ceiling, but >100k merchants in 2026 is enough to build a serious
   business. Expansion requires becoming a Tech Provider, which collides
   with the no-KYC stance (v2 question).

## Roadmap

### v1 (target 12–14 weeks)

- Klaviyo OAuth integration via Tech Partner API
- 3–5 pre-built JSON-driven Satori image templates with named variable
  slots, designed for WhatsApp/RCS/MMS header media
- **Klaviyo Events API orchestration** (the only viable send pattern —
  Klaviyo doesn't support inline URL token substitution): we POST a
  custom metric + image URL property; merchant's Klaviyo flow listens
  and triggers
- Klaviyo Flows API scaffolding during merchant onboarding (auto-create
  the flow + WhatsApp template definition that listens for our metric)
- Shopify cart-abandoned trigger as the demo use case
- Image-only renderer (no GIF, no video)
- R2 CDN with `(template, variable_hash)` cache key
- Meta-side template submission flow: merchant reviews and submits the
  Klaviyo-scaffolded template through Klaviyo's UI; Klaviyo handles the
  Meta approval queue

### v1.5 (target 18–20 weeks)

- GIF rendering
- Deeper webhook-orchestrated flow templates beyond cart abandoned
  (post-purchase, win-back, broadcast personalization)
- Visual canvas template editor (optional — depends on user feedback)

### v2 (target 6–9 months)

- Video on landing pages (not in messages — email/WA/RCS clients don't
  reliably support inline video)
- Add Postscript and/or Attentive as secondary BYO adapters (need
  separate TOS verification)
- Begin Meta Tech Provider application IF we decide to absorb the KYC
  burden for net-new WA merchants

### Explicitly NOT in roadmap

- Wati / Interakt / AiSensy / Sinch adapters — blocked by TOS; revisit
  only if partner-program approval lands with written sign-off
- Reseller mode / Embedded Signup — conflicts with no-KYC stance
- HubSpot / Zoho integration — separate product, different buyer, defer

## Next Steps

### Week 1 (parallel, no app code yet)

1. **Renderer spike**: 50-line Cloudflare Worker using `workers-og`,
   render a 1200×628 PNG, measure cold-start latency end-to-end
2. **Shopify dev store**: install scaffold app, subscribe to webhooks,
   start drafting Protected Customer Data Level 2 application
3. **Customer discovery**: 5–10 calls with Shopify+Klaviyo merchants —
   validate that "personalized images in WA/RCS/MMS via Klaviyo flows"
   is a real pain they'd pay for

**Note**: Klaviyo Tech Partner Program is NOT required for v1.
The merchant pastes their own Klaviyo API key into our app; Klaviyo's
public API Terms explicitly anticipate this 3rd-party integration
pattern. We have no contractual relationship with Klaviyo in v1 — we
just authenticate as the merchant when calling their API. Tech Partner
status becomes optional v1.5/v2 work for OAuth + marketplace
distribution + co-marketing, not a launch dependency.

### Weeks 2–8 (build v1)

5. Klaviyo OAuth + Events API integration
6. Template gallery (3–5 abandoned-cart-focused designs)
7. URL-only mode shipped first; webhook-orchestrated mode shipped second
8. Submit Protected Customer Data application
9. Internal alpha with 1–2 friendly merchants

### Weeks 9–14 (beta + launch)

10. Closed beta: 5–10 Klaviyo+Shopify merchants. Measure attributable
    lift in WA/RCS engagement.
11. Shopify App Store submission once Protected Customer Data approval
    + Klaviyo Tech Partner status both land
12. Public launch

### Decision points

- **If customer discovery surfaces minimal WA/RCS demand** (vs. demand
  for richer email/MMS personalization): re-evaluate whether we should
  compete with Movable Ink in the MMS+email lane via Klaviyo, or pivot
  to the HubSpot personalized-email-image lane (separate validation work
  showed it's empty on the Marketplace, but not in scope of this plan).

## Open Questions

1. Engineering capacity (solo, or small team?) — affects 12-week timeline
   reality
2. Klaviyo Tech Partner contact — any existing relationship, or applying
   cold?
3. Budget for validation spike + beta (Cloudflare Paid + Klaviyo Test
   account + Shopify Partner dev store ≈ ~$50–200 across first 30 days)
4. Brand / naming — defer

---

*Plan reflects converged direction as of brainstorm session. The original
Wati-first plan in this file's prior version was structurally blocked by
Wati TOS §4.4 and §4.1 — Klaviyo emerges as the only major messaging
provider where BYO-API-key is legally permitted AND commercially proven
on Shopify. Updated after BYO-BSP TOS audit verified the architectural
gap on Shopify (BYO-WhatsApp-BSP is mostly TOS-blocked, not unattempted)
and after architectural-pattern survey confirmed BYO-Klaviyo is
extensively validated (Loox, Judge.me, Smile.io, et al. operate exactly
this shape at scale).*
