# ADR-002: Lemon Squeezy as Merchant of Record over Stripe direct

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable will sell digital products (CV builder, cover letter generator, ECA concierge) and recurring subscriptions (Settlement Pro), with affiliate revenue layered on top. Customers will pay primarily in EUR and CAD, with a long tail of other currencies as francophone newcomers come from many countries. The business is operated solo from Turkey until late 2026, then from Canada once permanent residence is activated.

A payment provider must be chosen before the first paid feature ships. The choice has to handle three things at once: collect money from international customers, manage tax compliance across EU VAT, Canadian GST/HST, US sales tax, and assorted other jurisdictions, and remain operationally tractable for one developer with no accountant on staff.

The two viable architectures are: a Payment Service Provider (PSP) where the seller of record is Érable itself and tax compliance is the seller's responsibility, or a Merchant of Record (MoR) where the platform becomes the legal seller, collects taxes, files them, and pays out a net amount.

This decision is needed now because every product page, every subscription flow, every receipt template depends on it.

## Decision Drivers

- Operational accessibility from Turkey without setting up a foreign legal entity.
- Tax compliance across multiple jurisdictions handled by the platform, not by the founder.
- Native support for both one-time purchases and subscriptions in a single integration.
- Reasonable migration path to a more cost-efficient setup once the business is incorporated in Canada.
- No career signal driver here. Payment integration code is not what gets noticed in interviews.

## Considered Options

- A. Lemon Squeezy (MoR, owned by Stripe since 2024)
- B. Stripe direct (PSP, founder is seller of record)
- C. Paddle (MoR, independent)
- D. Polar (MoR, developer-focused challenger)

## Decision Outcome

Chosen option: **A. Lemon Squeezy**, because it is the only option that combines MoR tax handling, immediate operational accessibility from Turkey without offshore incorporation, and a migration path that stays inside the Stripe ecosystem once the business moves to Canada.

### Consequences

- Good: tax compliance for EU VAT, Canadian GST/HST, US sales tax, and other jurisdictions is handled end to end. No accountant needed for cross-border tax filing during the MVP.
- Good: subscriptions, one-time products, license keys, customer portal, and webhooks are all available from a single API.
- Good: ownership by Stripe since 2024 means the future migration to Stripe Managed Payments (currently early access) stays within the same group, with documented migration paths likely to mature over 2026.
- Good: signup works from Turkey without setting up a US LLC or Estonia e-Residency. Time to first sale is days, not weeks.
- Bad: effective fee rate for Érable's profile lands around 7-8 percent, not the headline 5 percent + 0.50 USD. The breakdown: 5% base + 0.50 USD per transaction, plus 1.5% for international transactions (which applies to all sales since Érable targets Canada and Europe), plus 0.5% for subscription payments, plus 1% on payouts to non-US bank accounts. Affiliate-referred sales add another 3%.
- Bad: the cost gap with Stripe direct (roughly 2.9% + 0.30 USD plus VAT services) widens with revenue. At higher MRR, the MoR convenience starts costing real money compared to hiring fractional VAT compliance.
- Bad: less control over the checkout UX than Stripe Checkout or Stripe Elements would offer. We accept the trade-off because customization is not where MVP-stage value is created.
- Bad: payouts to a Turkish bank are slower and pricier than payouts to a US or EU bank; this becomes a non-issue once banking moves to Canada.

## Pros and Cons of the Options

### Option A: Lemon Squeezy

- Good: full Merchant of Record. Sales tax, VAT, fraud, chargebacks, and dispute handling are platform responsibilities, not founder responsibilities.
- Good: signup works from Turkey with a Turkish bank account. No offshore incorporation required.
- Good: native support for one-time purchases, subscriptions, license keys, and a customer portal in one API surface.
- Good: now part of Stripe (acquired 2024), which de-risks the long-term commitment and aligns with Stripe Managed Payments as a future migration target.
- Bad: effective fee rate ~7-8% for Érable's mix of international, subscription, and possibly affiliate-driven sales.
- Bad: less checkout customization than Stripe direct.
- Neutral: brand recognition is weaker than Stripe's, but the checkout page is clean and trustworthy.

### Option B: Stripe direct

- Good: lowest fees of the four options, around 2.9% + 0.30 USD with predictable surcharges for currency conversion and international cards.
- Good: best-in-class developer experience, deepest documentation, largest integration ecosystem.
- Good: maximum control over checkout UX, subscription logic, and dunning flows.
- Bad: not the seller of record. Érable would be liable for VAT registration and filing in every jurisdiction where it crosses thresholds. This is operationally unrealistic for a solo founder without a CPA.
- Bad: Stripe direct requires a legal entity and bank account in one of its supported countries. Turkey is not supported. Practical access from Turkey requires a US LLC or Estonia e-Residency setup with offshore banking — 500 to 1500 USD upfront plus ongoing dual-jurisdiction tax compliance. Disproportionate for a pre-revenue MVP.
- Neutral: becomes the natural target once Érable is incorporated in Canada and revenue justifies a fractional CPA for VAT.

### Option C: Paddle

- Good: full Merchant of Record like Lemon Squeezy, with longer history and a strong reputation among SaaS sellers.
- Good: tax handling and compliance are mature.
- Good: accessible from Turkey.
- Bad: fees comparable to or slightly higher than Lemon Squeezy depending on volume.
- Bad: integration surface is heavier and less developer-friendly. Documentation feels enterprise-oriented.
- Bad: no Stripe ecosystem alignment, which weakens the migration path when Érable moves to Canada.
- Neutral: a defensible alternative on technical merit; loses on ecosystem alignment.

### Option D: Polar

- Good: Merchant of Record with a developer-first, open-source-friendly positioning that aligns with Érable's brand.
- Good: lower headline fees than Lemon Squeezy and Paddle on some plans.
- Good: rapid iteration, modern API design.
- Bad: smallest and youngest of the four. Tax compliance coverage and operational maturity have less track record. For a project that intends to invoice paying customers in week 11, betting on the youngest player adds unnecessary risk.
- Bad: smaller community, fewer integration examples, less battle-testing on edge cases like chargebacks and refunds.
- Neutral: worth revisiting in 12-18 months if it consolidates its position.

## More Information

- Related ADRs: none direct. ADR-007 (privacy architecture) interacts indirectly because payment data flows through the MoR and never touches our database.
- References:
  - Lemon Squeezy fee structure documentation, current as of May 2026.
  - Stripe global availability page; Turkey is not in the supported countries list.
  - Stripe acquisition of Lemon Squeezy, August 2024.
  - Stripe Managed Payments early access announcement, January 2026.
- Revisit trigger: revisit this decision if any of the following occur:
  - Stripe Managed Payments reaches general availability with a documented migration path from Lemon Squeezy. This is the expected end-state for Érable.
  - Érable is incorporated in Canada with a Canadian bank account, making Stripe direct viable, AND MRR justifies fractional CPA fees for in-house VAT compliance (rough threshold: 10k USD MRR sustained for two quarters).
  - Lemon Squeezy materially changes its terms or pricing, or its integration with Stripe degrades the product.
  - A regulatory change in Turkey or Canada makes the current setup impractical.
