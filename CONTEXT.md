# Personalized Media — Context

The domain language for the Shopify-based personalized rich-messaging
product. Terms below have precise meanings in this project and should
be used consistently in code, copy, and conversation.

## Language

### Messaging stack

**BSP (Business Solution Provider)**:
A Meta-authorized vendor that operates WhatsApp Business API access on
behalf of brands. Wati, Interakt, AiSensy, 360dialog, Klaviyo (since
becoming Tech Provider), Twilio, Sinch.
_Avoid_: "WhatsApp provider," "WA partner."

**RBM Partner (RCS Business Messaging Partner)**:
A Google-authorized vendor that operates RCS Agent registration and
sending on behalf of brands. Sinch, Twilio, Klaviyo, Vibes, Infobip.
_Avoid_: "RCS provider," "RCS aggregator."

**WABA (WhatsApp Business Account)**:
The Meta-verified business identity that owns the right to send
WhatsApp messages. Lives in Meta Business Manager. One per merchant.
_Avoid_: "WhatsApp account," "WA business."

**RCS Agent**:
The Google-verified brand identity for RCS — brand name, logo,
verified-business badge, sender number. One per merchant.
_Avoid_: "RCS account," "RCS sender."

**Tech Provider**:
Meta's term for an ISV that holds a direct contractual relationship
with Meta and can onboard merchants' WABAs via Embedded Signup. Klaviyo
is a Tech Provider; **we are not** (and won't be in v1).
_Avoid_: "Meta partner," "ISV."

**Embedded Signup**:
Meta's white-label business-verification flow that a Tech Provider
hosts inside their own app UI. We don't use this in v1.

### Our product

**Renderer**:
Our service that converts a (template, variables) tuple into a
personalized PNG, hosted at a stable URL keyed by `(template_id,
variable_hash)`.
_Avoid_: "image service," "personalization engine."

**Template**:
A JSON-driven Satori definition with named variable slots
(e.g. `firstName`, `productImage`, `price`). The merchant picks one
from our gallery during onboarding; we don't ship a visual editor in
v1.
_Avoid_: "design," "layout."

**Klaviyo Template**:
**Different from our Template.** This is the WhatsApp/RCS message
template registered through Klaviyo's UI and approved by Meta (for
WhatsApp) or Google (for RCS). Our app may scaffold these via Klaviyo's
Flows API during merchant onboarding.

**Header media**:
The image (or video / document) at the top of a WhatsApp/RCS message
template. The URL of the header image can vary per recipient — this is
the personalization vector we hook into.

**Variable hash**:
A deterministic hash of the personalization input used as part of our
PNG URL path. Same inputs → same URL → Meta cache hit. Different
inputs → different URL → new render.

### Integration architecture

**BYO-BSP (Bring Your Own BSP)**:
Our product model. The merchant has an existing relationship with a
messaging provider (Klaviyo in v1) and brings their credentials. We
add a layer; we don't replace.
_Avoid_: "passthrough integration," "BSP-agnostic."

**Reseller model** (rejected for v1):
The alternative where we onboard merchants to WhatsApp/RCS ourselves
via Embedded Signup, charge them for messages, and hold the BSP
relationship. Rejected because it requires KYC facilitation.
_Avoid_: "white-label send," "managed WA."

**Events API orchestration**:
The send pattern we use. Our backend POSTs a custom metric +
properties (including the personalized image URL) to Klaviyo's Events
API; a merchant-configured Klaviyo flow listens for that metric and
triggers the message send.

**A/B trial**:
A 30-day split-test that runs automatically at merchant install.
Abandoned-cart events are deterministically split 50/50 by recipient:
half receive our personalized template (**Trial Group A**), half
receive the merchant's existing baseline (**Trial Group B**). After
30 days, the merchant sees side-by-side recovery rate + revenue and
decides whether to upgrade to a paid tier.

**Baseline template**:
The merchant's pre-existing cart-recovery WhatsApp template, or the
default Klaviyo plaintext message if none exists. Group B in the A/B
trial. We never modify this — we just reference it for comparison.

**Personalized template**:
Our scaffolded WhatsApp template with our rendered PNG as the header
media. Group A in the A/B trial.

### Buyer + market

**Klaviyo merchant**:
A Shopify store using Klaviyo as their email/SMS/WhatsApp/RCS provider.
Our target ICP. ~117k brands on Shopify run on Klaviyo per Klaviyo's
public disclosures.

**ICP (Ideal Customer Profile)**:
Shopify Plus or growth-stage merchant, Western markets, English
operating language, ~$1M–$50M GMV, already on Klaviyo's paid plan,
actively using their WhatsApp/RCS channels (or interested in
activating).

**Primary buyer persona**: **E-commerce Manager** at the merchant — a
Klaviyo power-user who runs flows daily, has budget authority for
$50–$200/mo tools, and treats us as "a Klaviyo enhancement." They are
neither the founder (smaller stores, $20–49/mo comfort zone, higher
churn risk) nor the CMO (mid-market+, doesn't make individual
tool-level decisions). The product is shaped for this persona; others
buy incidentally.

**Primary JTBD** (Job-to-be-done): **Recover more abandoned carts.**
The E-com Manager hires us to lift their existing cart-recovery
recovery-rate KPI by rendering personalized banners into the
WhatsApp/RCS messages they already send via Klaviyo. Other JTBDs
(brand polish, channel activation, competitive differentiation) are
adjacent — we serve them incidentally but the product is shaped for
cart recovery specifically: cart-shaped templates, cart-recovery
benchmarking, cart-recovery onboarding flow scaffolded by default.

## Relationships

- A **Merchant** owns one **WABA** and one **RCS Agent** through their
  **BSP** (Klaviyo in v1)
- Our **Renderer** produces a **PNG** at a URL the **BSP** fetches at
  send time and serves as **Header media** to the recipient
- A **Klaviyo Template** references **Header media** via a property
  variable resolved at send time from a **Klaviyo Event** that our
  backend POSTed
- A **Template** (ours) is a Satori definition that produces personalized
  PNGs; a **Klaviyo Template** is a Meta-approved message format. These
  are distinct concepts.

## Example dialogue

> **Dev:** "When the merchant connects their **Klaviyo** account, do we
> create the **Klaviyo Template** automatically?"
>
> **PM:** "Yes — we use Klaviyo's Flows API to scaffold one. But
> approval still goes through Klaviyo to Meta, so the merchant clicks
> 'submit for approval' inside Klaviyo's UI; that's not something we
> can automate around."
>
> **Dev:** "And the per-recipient personalization?"
>
> **PM:** "The **Klaviyo Template**'s header image field is mapped to
> `{{ event.image_url }}`. We POST that URL via the **Events API** when
> Shopify fires the cart-abandoned webhook. The **Renderer** has
> already produced the PNG by then. The **BSP** (Klaviyo) is the only
> party that calls Meta."

## Flagged ambiguities

- "Template" — initially used for both our Satori definitions and
  Klaviyo's message templates. Resolved: **Template** = ours,
  **Klaviyo Template** = Meta-approved message format. Always
  disambiguate in writing.
- "Partner" — initially conflated with **Tech Partner** (Klaviyo's
  formal program) and "integration partner" (informal positioning).
  Resolved: **Tech Partner** = the Klaviyo legal program (not v1);
  "partner" alone = avoid; use "integration" or "BSP" instead.
- "Customer" — could mean the **Merchant** (our buyer) or the merchant's
  end-customer (the recipient of the message). Resolved: **Merchant**
  for our buyer; "recipient" for end-customer.
