# Built-in A/B trial as the attribution model

On install, the merchant's abandoned-cart traffic is split 50/50: half
the events use our personalized template, half use the merchant's
existing baseline template. After 30 days, we show side-by-side recovery
rate and recovered revenue. The merchant decides whether to stay (and
upgrade to a paid tier) based on real numbers, not on our claimed lift.

## Considered Options

- **No attribution UI in v1** — fastest to ship; gambles on month-2
  retention. Rejected: E-com Manager persona measures everything; "trust
  us, it's working" doesn't survive the second invoice.
- **Klaviyo native analytics + before/after comparison** — pulls recovery
  rate from Klaviyo's API for our flow vs. merchant's prior baseline.
  Cheap but confounded (seasonality, traffic mix, other product changes).
  Honest about limitations but weak as a retention story.
- **Multi-variate dashboard** — recovery rate by template / segment /
  time-of-day. Overbuilds for MVP; defer to v1.5+ if signal emerges.
- **Selected: Built-in A/B trial for first 30 days** — only path that
  honestly isolates our contribution; aligns trial UX with attribution
  measurement; turns the merchant's natural skepticism into the product's
  retention engine.

## Consequences

- Onboarding scaffolds **two** Klaviyo flows per merchant (or one flow
  with conditional logic, depending on what Klaviyo's Flows API allows
  — see [ADR-0004](0004-events-api-orchestration.md))
- Our backend must split Shopify cart-abandoned events 50/50 deterministically
  per recipient (so the same shopper is consistently in one group across
  multiple sessions)
- Trial outcome dashboard pulls per-flow recovery + revenue metrics from
  Klaviyo's API
- Pricing model needs a "trial → paid" transition (free during trial,
  paid after). Trial expiry triggers conversion event.
- We can't claim a fixed "X% lift" headline number — each merchant's
  trial result is their own. The marketing site shows aggregate ranges
  ("trial merchants see 8–22% recovery rate lift") but the merchant's
  decision is based on their own A/B.
- If a merchant's trial shows no lift, our retention story is broken
  for them specifically. Acceptable cost — better to lose merchants who
  truly don't benefit than to retain them on false claims.
