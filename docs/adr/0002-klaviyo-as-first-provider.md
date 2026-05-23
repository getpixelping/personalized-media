# Klaviyo as v1's only BSP, not Wati

The original brainstorm direction was BYO-Wati (India/SEA SMB Shopify
merchants). That direction was killed by a BSP TOS audit (see
[research/02-bsp-tos-audit.md](../research/02-bsp-tos-audit.md)) which
found that Wati's §4.4 prohibits credential disclosure to third parties
and §4.1 prohibits transferring the Solution to third parties. Most
major WhatsApp BSPs (Interakt, Sinch-WhatsApp) have similar or stricter
language. **Klaviyo is one of only two BSPs (along with 360dialog)
where the BYO-API-key pattern is explicitly permitted by their API
Terms** — and Klaviyo is the only one whose pattern is already
extensively proven on the Shopify App Store (Loox, Smile.io, Judge.me,
Stamped all use BYO-Klaviyo at scale).

## Consequences

- The architectural pattern is commercially de-risked — many high-revenue
  Shopify apps use it
- One Klaviyo integration unlocks all our target channels (WA + RCS +
  MMS + SMS) — no multi-adapter complexity in v1
- We compete inside Klaviyo's installed base. Movable Ink covers
  Klaviyo's email + MMS but not WA/RCS — that gap is our wedge
- Wati/Interakt are deferred to v1.5+ contingent on Technology Partner
  Program approval with written sign-off (months-long BD process,
  uncertain outcome)
