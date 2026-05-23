# Entity requirements + individual→company transition

**Question**: Do Shopify and Klaviyo require a registered company
entity for partners/integrations? And what's the path to convert from
individual to a registered entity later?

## Shopify — individual-friendly at every gate

| Gate | Individual accepted? | Source |
|---|---|---|
| Partner signup | ✅ "Partner means an individual or entity" | [Shopify Partner Program Agreement](https://www.shopify.com/partners/terms) |
| App Store listing | ✅ No entity rule for developer-name | [App requirements checklist](https://shopify.dev/docs/apps/launch/app-requirements-checklist) |
| Protected Customer Data Level 2 review | ✅ Reviews **technical practices**, not entity papers | [Protected Customer Data](https://shopify.dev/docs/apps/launch/protected-customer-data) |
| Payouts (via Hyperwallet) | ✅ Individual W-9 / W-8BEN accepted | [Manage your payout method](https://help.shopify.com/en/partners/manage-account/manage-payouts-invoices/payout-method) |

**Quoted clause** (Shopify Partner Program Agreement):
> "Partner means an individual or entity that has agreed to the terms
> of this Agreement and participates in the Shopify Partner Program."
> "If the Partner is an individual, you must be the older of 18 years
> or at least the age of majority in the jurisdiction where you reside."

## Klaviyo — individual-friendly in the legal text

**Quoted clause** (Klaviyo Technology Partner Agreement):
> "This Agreement is between Klaviyo, Inc. and **you or the legal entity
> entering into this Agreement** ('Partner,' or 'you,' or 'your')."

App Listing Requirements: gates are technical (TLS 1.2+, MFA, OAuth) +
traction-based (minimum 5 production installs before listing). No
entity rule.

**Tech Partner Program is OPTIONAL for v1** — see
[ADR-0003](../adr/0003-public-api-not-tech-partner.md). Using Klaviyo's
public API with the merchant's BYO key doesn't require any agreement
with Klaviyo at all.

## Individual → company transition path

### Klaviyo §10(k) Assignment clause (verified):
> "You will not assign or transfer this Agreement, including any
> assignment or transfer by reason of merger, reorganization, sale of
> all or substantially all of its assets, change of control or
> operation of law, **without our prior written consent**."

**Practical implication**: only applies if we signed the Tech Partner
Agreement. Since we don't (per ADR-0003), §10(k) is moot. No
notification to Klaviyo is needed when the founder incorporates.

### Shopify transition (not directly verified via WebFetch — based on prior knowledge):
- Partner Account business details (legal name, tax form) typically
  updatable in-place via dashboard settings
- App ownership transfer between Partner Accounts is a documented
  feature, typically routed through Partner support
- Installs / reviews / app ID are usually preserved across transfer

### Standard playbook (US founder pattern)

1. **Months 1–6**: Ship MVP as individual. Personal Shopify Partner
   Account + W-9 individual.
2. **Month ~6 (PMF signal)**: Form entity. Most US-resident founders
   use Delaware/Wyoming LLC via Stripe Atlas / Clerky (~$500–$1,500).
3. **Immediately after**: Execute IP-assignment agreement
   (individual → LLC). §351 contribution in US tax. Templates from
   Cooley GO / Stripe Atlas.
4. **Update Shopify Partner Account** with new entity / W-9 business.
5. **Update bank/payout destination** to corp's business account.
6. **No Klaviyo step needed** (we never signed their agreement).
7. **Update privacy policy + ToS** on app site referencing the new
   entity as the data controller.
8. **Update App Store listing** "Developed by" name if it changes.

**Total cost**: $500–$2,000 in entity formation + IP-assignment legal.
**Timeline**: 3–6 weeks calendar.

## Sources

- [Shopify Partner Program Agreement](https://www.shopify.com/partners/terms)
- [Klaviyo Technology Partner Agreement](https://www.klaviyo.com/legal/technology-partner-agreement)
- [Klaviyo App Listing Requirements](https://developers.klaviyo.com/en/docs/klaviyo_app_listing_requirements)
- [Shopify Protected Customer Data](https://shopify.dev/docs/apps/launch/protected-customer-data)
