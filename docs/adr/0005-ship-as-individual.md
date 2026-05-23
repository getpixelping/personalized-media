# Ship v1 as an individual; incorporate post-PMF

Both Shopify and Klaviyo accept individual sole-proprietors as
Partners/integrators (verified — see
[research/05-entity-and-transition-policies.md](../research/05-entity-and-transition-policies.md)).
Shopify's Partner Program Agreement explicitly defines "Partner" as
"an individual or entity"; Klaviyo's API Terms permit BYO-key
integrations without any partner-program relationship. Founder ships
MVP as an individual, validates product-market fit with 5–10 paying
merchants, then incorporates (likely US LLC via Stripe Atlas / Clerky)
in month 4–6.

## Considered Options

- **Register entity upfront, sign all agreements as the entity** —
  cleaner long-term posture but adds $200–$1,500 + 1–2 weeks before
  any other work starts; ties up cash that could fund development;
  premature given no validated demand
- **Selected: individual now, entity post-PMF** — no incorporation
  cost during the high-risk validation phase; aligns with standard
  indie SaaS founder pattern (Stripe Atlas / Clerky exist precisely
  for this transition)

## Consequences

- Personal liability for app-related claims during v1 — capped only
  by indemnity clauses in Shopify's Partner Agreement, not isolated
  from personal assets
- US tax: app revenue flows to Schedule C until incorporation
- When incorporating later, execute an IP-assignment agreement
  transferring code, brand, customer contracts to the new entity
  (standard §351 contribution in US tax). Stripe Atlas / Clerky
  templates available.
- No Klaviyo notification needed during incorporation — we never signed
  Klaviyo's Tech Partner Agreement, so there's nothing to assign
- Shopify Partner Account business details can be updated in-place
  when entity is formed (no full transfer needed)
