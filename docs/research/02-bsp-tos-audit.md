# BSP TOS audit — BYO-API-key permission across providers

**Question**: For each major messaging provider, does their TOS permit
a Shopify app to use the merchant's API credentials to send on the
merchant's behalf?

**Why this matters**: An earlier architectural-pattern survey
([research/03](03-shopify-byo-pattern-survey.md)) found that BYO-WhatsApp-BSP
is unprecedented on the Shopify App Store. We needed to know whether
that's an unattempted greenfield or a structural prohibition.

**Finding**: Mostly structural prohibition. Klaviyo is the clean exception.

## Ranking (most → least permissive for BYO architecture)

| Rank | Provider | Verdict | Key clause |
|---|---|---|---|
| 1 | **Klaviyo** | ✅ Explicitly permitted | API Terms anticipate third-party integrations using customer keys; scoped keys + OAuth are designed for this |
| 2 | 360dialog | ✅ ISV-friendly, but canonical pattern is Embedded Signup (= KYC) | Tech Provider program is the marketed flow |
| 3 | Twilio | 🟡 Allowed in spirit; canonical pattern is subaccount hierarchy | ISV partner program exists |
| 4 | AiSensy | 🟡 TOS silent | Needs sales-rep confirmation |
| 5 | Interakt | 🔴 Prohibited by literal text | §5(a): "license, sublicense, sell, transfer, distribute or otherwise commercially exploit or make available to any third party the Solution" |
| 6 | Wati | 🔴 Prohibited absent Technology Partner Program approval | §4.4: "ensure Administrative Users do not...disclose any usernames, passwords or other access credentials." §4.1: no transfer to third party |
| 7 | Sinch (WhatsApp) | 🔴 Hard block | §2.6: "You agree that you will **not disclose the certificate** required to access the WhatsApp Business Client to any ISV [or] other third parties" |

## Structural cause

The reason no Shopify app currently does BYO-WhatsApp-BSP isn't lack of
imagination — it's that **Meta's Tech Provider model treats the BSP as
the regulated credential holder**, so individual BSPs inherit a strict
no-credential-sharing posture downstream.

Meta's canonical ISV pattern is **Tech Provider + Embedded Signup** —
reseller-shape, requires KYC facilitation, explicitly rejected for v1
(see [ADR-0001](../adr/0001-byo-bsp-not-reseller.md)).

Klaviyo sidesteps this because **Klaviyo is itself a registered Tech
Provider**. They hold the Meta relationship; merchants' WABAs are
registered through Klaviyo; our app uses Klaviyo's API — Meta has
already blessed Klaviyo as a legitimate sender, and Klaviyo's own API
Terms permit third-party integrations like ours. We sit on a legal
pyramid, not as a credential-sharer.

## Sources

- [Wati Terms & Conditions](https://www.wati.io/terms-and-conditions/)
- [Interakt Terms of Service](https://www.interakt.shop/terms-of-service/)
- [Klaviyo API Terms of Use](https://www.klaviyo.com/legal/api-terms)
- [Klaviyo API Key Management](https://help.klaviyo.com/hc/en-us/articles/115005062267)
- [AiSensy ToS](https://aisensy.com/tos)
- [360dialog Tech Provider Program](https://docs.360dialog.com/partner/get-started/tech-provider-program/tech-provider-program-info)
- [Twilio ToS](https://www.twilio.com/en-us/legal/tos)
- [Sinch WhatsApp ISV Pass-On Supplemental Terms](https://www.sinch.com/en-in/whatsapp-isv-pass-services-supplemental-terms-and-conditions/)
