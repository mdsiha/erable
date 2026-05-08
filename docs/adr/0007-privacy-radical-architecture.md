# ADR-007: Privacy-radical architecture

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable serves French-speaking newcomers to Canada — a population that is, by demographic and cultural disposition, particularly attentive to how their personal data is handled. Many arrive from jurisdictions (France, Belgium, North Africa) where GDPR awareness is high, and from communities where trust in American cloud products has been eroded by a decade of public data scandals. Several of Érable's planned features depend on processing sensitive material: official letters from IRCC, pay slips for credential recognition, banking documents for newcomer account applications, scanned diplomas for ECA workflows.

A privacy posture must be chosen and codified before any feature that touches user documents ships. The posture cannot be retrofitted; once data flows are established, changing them costs more than designing them right the first time.

Three jurisdictions apply simultaneously. PIPEDA (Personal Information Protection and Electronic Documents Act) governs federally. Loi 25 (Québec, in force since 2023) is stricter than PIPEDA on consent, breach notification, and data residency. GDPR applies to any European user who reaches the product, which is a non-trivial fraction of the francophone target audience. A single coherent posture that satisfies the strictest of the three is operationally simpler than three parallel compliance regimes.

Beyond compliance, a deeper question shapes the decision. Most SaaS products publish a privacy policy that users must trust on faith. Érable's repository is public under MIT license. The privacy promises in PRIVACY.md can be verified against the code that implements them. This turns privacy from a marketing claim into an auditable property — a rare differentiator for a product in this space.

## Decision Drivers

- Trust and adoption with the target audience. Francophone newcomers to Canada are sensitive to privacy, and a verifiable posture is a marketing advantage as much as an ethical one.
- Compliance with PIPEDA, Loi 25, and GDPR simultaneously. Designing to the strictest regime is simpler than running three parallel compliance regimes.
- Verifiability. The codebase is public; privacy claims must hold up to inspection. "Trust us" is replaced by "read the code".
- Operational simplicity. Less data stored means less attack surface, fewer breach scenarios, less regulatory risk, and lower storage and backup cost.
- No career signal driver. This decision is ethical and business-driven; framing it as career-relevant would weaken the ADR.

## Considered Options

- A. Privacy-radical posture
- B. Privacy-standard SaaS posture
- C. Compliance-minimal posture

## Decision Outcome

Chosen option: **A. Privacy-radical posture**, because it aligns with the target audience's expectations, simplifies multi-jurisdiction compliance, becomes a verifiable differentiator under the open-source license, and reduces the operational and reputational risk surface to near-zero for the categories of data we never collect.

The posture is implemented through five coupled sub-decisions, each documented below with the alternative considered and rejected.

### Sub-decision 1: User-submitted documents are never persistently stored

Documents uploaded for OCR or analysis (IRCC letters, pay slips, diplomas) are processed in memory or in an ephemeral MinIO bucket with a 60-second lifecycle policy. The application explicitly deletes the object after processing completes; the lifecycle policy is the safety net for failures or crashes. No copy of the document, no derived image, no intermediate file, and no PII-bearing extraction output is persisted beyond the request lifecycle.

Anonymized structural learning events (document category from a closed taxonomy, extraction quality metrics, interpretation references, bucketed user profile) are recorded separately in a dedicated `learning_events` store. These events contain no PII, no document content, and no individually identifying field combinations. The architecture and guarantees of this learning store are documented in detail in ADR-008.

Alternative rejected: short-term encrypted storage (24-72 hours) for user convenience and debugging. Rejected because the marginal UX benefit (re-upload avoidance for a small fraction of users) does not justify the regulatory exposure, the breach risk, the storage cost, and the loss of the verifiable "we never store your documents" claim.

### Sub-decision 2: OCR runs locally by default; cloud OCR is opt-in per upload

Tesseract 5 with French and English language packs runs on the production VPS. It handles printed text well, which covers the majority of newcomer documents. For handwritten material or documents Tesseract cannot read confidently, the user is presented with an explicit, per-upload choice to send the document to Claude Vision. The choice is opt-in, never opt-out; there is no global setting that pre-authorizes cloud OCR for future uploads.

Alternative rejected: Claude Vision as the systematic OCR engine. Rejected because it sends document content to Anthropic's infrastructure for every upload, even when local OCR would suffice. The convenience does not justify the loss of the "documents stay on Canadian infrastructure unless you explicitly choose otherwise" claim.

### Sub-decision 3: Personally identifiable information never enters the logs

The structured logger (structlog) is configured with a redaction layer that strips email addresses, names, addresses, phone numbers, and document content from log payloads. Log levels DEBUG and INFO never receive PII even in development. Stack traces and error contexts are scrubbed before being written or sent to Sentry.

Alternative rejected: verbose logs with PII, masked at query time. Rejected because the masking approach assumes nothing leaks, which is the wrong default. Logs are copied, archived, sent to error trackers, and read by humans during incidents. Each of these is a leak vector. Stripping at write time eliminates the class of risks entirely.

### Sub-decision 4: Affiliate tracking only on authenticated users

Affiliate links (banking newcomer programs, telecom plans, etc.) are tracked only when an authenticated user clicks them. The tracking ties the click to the user account through a server-side mapping. No cookie-based fingerprinting, no anonymous tracking pixels, no third-party tracking scripts.

Alternative rejected: anonymous fingerprint-based tracking, the SaaS standard. Rejected because it processes data from users who have not consented, and because the marginal revenue uplift (catching anonymous referrals) does not justify the conflict with the verifiable privacy posture.

### Sub-decision 5: Document deletion is a hard 60-second contract

Every document upload is associated with a deletion deadline 60 seconds after upload. The MinIO bucket's lifecycle policy enforces this at the storage layer; the application enforces it at the request handler layer; an ARQ background job sweeps any orphans older than 60 seconds every 5 minutes. Three layers, all converging on the same guarantee.

Alternative rejected: best-effort deletion with no formal deadline. Rejected because best-effort is what every SaaS product claims and what no user can verify. A hard contract, enforced at three layers, is what makes the privacy promise auditable.

### Consequences

- Good: trust posture aligned with the target audience. The "we never store your documents" claim is short, verifiable, and resonant.
- Good: multi-jurisdictional compliance simplified. Designing to the strictest of PIPEDA, Loi 25, and GDPR yields a single posture that satisfies all three without a per-region implementation.
- Good: breach surface for sensitive document content is effectively zero. We cannot leak what we do not store.
- Good: open-source-license verifiability turns the privacy promise into an auditable property. This is a marketing asset and a credibility asset simultaneously.
- Good: operational simplicity. No long-term encryption-at-rest schemes for user documents, no key rotation for those schemes, no data subject access request workflow for documents we never had.
- Bad: users cannot retrieve a previously analyzed document. If a user uploads an IRCC letter on day one and wants to see it again on day eight, they must re-upload. Acceptable for the use case; users who want personal document storage have many other tools for that.
- Bad: we cannot improve OCR or interpretation by analyzing real user documents. The local Tesseract setup will not benefit from fine-tuning on actual newcomer documents. This is mitigated, not eliminated, by the privacy-preserving learning architecture in ADR-008, which captures structural quality signals (extraction corrections, interpretation feedback) without storing source content. Some product improvements that strictly require source documents remain out of reach by design.
- Bad: the learning-events approach (ADR-008) is privacy-preserving by construction but requires ongoing discipline to avoid scope creep. Each new field added to the schema must be reviewed against the no-PII guarantee. The closed taxonomies must be maintained to prevent free-text leakage.
- Bad: anonymous-tracking-based affiliate revenue is not captured. The conversion attribution window only covers authenticated users. We accept the revenue gap as the cost of the posture.
- Bad: the "60-second deletion" contract requires real engineering work (three enforcement layers) and real testing (it must hold under failure modes). The cost is bounded but not zero.

## Pros and Cons of the Options

### Option A: Privacy-radical posture

- Good: aligns with target audience expectations and produces a marketing differentiator that competitors cannot easily replicate.
- Good: simplifies multi-jurisdiction compliance to a single coherent posture.
- Good: becomes a verifiable, auditable property under the open-source license. This is rare and valuable.
- Good: minimizes attack surface and breach risk for the categories of data we choose not to collect.
- Bad: small but real UX cost (no document history, re-uploads required).
- Bad: small but real product cost (no in-house OCR improvement loop).
- Bad: small but real revenue cost (no anonymous-tracking attribution).
- Neutral: requires discipline to maintain. Every new feature must be evaluated against the posture before it ships.

### Option B: Privacy-standard SaaS posture

- Good: maximally flexible. Documents can be stored, OCR can be cloud-first, telemetry can be rich, attribution can be complete.
- Good: easier to implement growth and product features that depend on retained data (recommendation engines, document history, behavioral analytics).
- Good: matches what most SaaS products do, so users are familiar with the model.
- Bad: incompatible with verifiable privacy claims. Privacy becomes a policy promise, not a code property.
- Bad: more compliance complexity in the GDPR/Loi 25 jurisdiction overlap.
- Bad: real breach surface for stored documents, with all the regulatory and reputational consequences that follow.
- Bad: contradicts the trust positioning that the target audience values most.
- Neutral: a defensible default for products whose users are not particularly privacy-sensitive. Not our shape.

### Option C: Compliance-minimal posture

- Good: strict adherence to the letter of PIPEDA, Loi 25, and GDPR. Legally defensible, no over-engineering.
- Good: lower implementation cost than option A.
- Bad: indistinguishable from option B in user perception. Users do not read privacy policies; they assume all SaaS products handle data the same way unless given a strong reason to believe otherwise.
- Bad: misses the marketing and trust upside of option A. Compliance is a floor, not a differentiator.
- Bad: still requires the operational machinery for data subject access requests, breach notification workflows, and consent management — none of which option A needs for the data it does not collect.
- Neutral: a reasonable default for a product without a privacy-sensitive audience. Not our shape.

## More Information

- Related ADRs:
  - ADR-002 (Lemon Squeezy as MoR) — payment data flows through the merchant of record and never enters our database, which is a privacy benefit consistent with this posture.
  - ADR-004 (auth from scratch) — rejected identity-as-a-service options largely on privacy grounds. This ADR formalizes that broader stance.
  - ADR-006 (pgvector over dedicated vector DB) — keeping vectors in our own PostgreSQL avoids sending document-derived embeddings to a third-party vector service.
  - ADR-008 (privacy-preserving learning architecture) — defines the `learning_events` store referenced in Sub-decision 1, and operationalizes service improvement within the posture set here.
  - Future ADR on telemetry and analytics — will operate within this posture (Plausible self-hosted, no third-party analytics scripts).
- References:
  - PIPEDA, full text and Office of the Privacy Commissioner of Canada guidance.
  - Loi 25 (Québec), Commission d'accès à l'information guidance.
  - GDPR articles relevant to data minimization (Article 5(1)(c)) and storage limitation (Article 5(1)(e)).
  - PRIVACY.md in the public repository, which documents these guarantees in user-facing language.
  - The data freshness pipeline in STACK_AND_CONVENTIONS.md, which complements this posture by ensuring the data we *do* serve is current and verifiable.
- Revisit trigger: revisit this decision if any of the following occur:
  - Recurring user requests for document history reach a volume that suggests the UX cost outweighs the trust benefit. Measure via support tickets and user research, not assumption.
  - A regulatory change makes a less strict posture both acceptable and operationally simpler.
  - A specific feature that materially serves users cannot be implemented without persistence (a concrete example, not a hypothetical), and the trade-off is judged worth making.
  - The learning architecture defined in ADR-008 fails to produce a useful improvement signal after 12 or more months of operation, suggesting that meaningful service improvement requires access to source content. At that point the trade-off is reopened, not foregone.
  - Competitive landscape shifts such that the verifiable privacy posture no longer differentiates, in which case the operational cost may no longer be justified.
