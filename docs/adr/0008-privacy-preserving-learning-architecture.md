# ADR-008: Privacy-preserving learning architecture

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable's strategic differentiation is contextual intelligence about Canadian administrative procedures, not raw OCR or document parsing. The product gets better when it understands, for a given user profile and a given administrative document, which interpretation is correct, which actions to recommend, and which paths lead to successful outcomes. Generic vision and language models do not have this knowledge; nobody does today; the data to build it does not exist in any public corpus.

ADR-007 commits to a privacy-radical posture: user-submitted documents are never persistently stored. Without further design, this commitment also forecloses the ability to learn from real usage. That foreclosure would, over time, make Érable indistinguishable from any wrapper around a generic model. The strategic moat would erode.

The decision needed is whether and how to capture a learning signal from real usage without storing user content, without retaining personally identifiable information, and without weakening the privacy claims that ADR-007 makes auditable. The decision must be made before any feature that touches documents or interpretations ships, because retrofitting privacy guarantees onto a data store that already exists is harder than designing them in.

A second-order goal sits behind the immediate one. The owner has stated explicitly that the option to train a domain-specific model later — not the commitment, but the option — must be preserved. This requires data of a particular shape: structured, labeled, voluminous, and free of PII. Not a corpus of documents.

## Decision Drivers

- The strategic intelligence of Érable resides in contextual understanding (situation → document → interpretation → action → outcome), not in OCR fidelity. The data architecture must capture that signal.
- ADR-007 privacy posture is non-negotiable. Any learning store must be defensible under the same audit standard: verifiable in code, no PII, no re-identification path.
- The option to train a domain-specific model in the future must be preserved without committing to it now. The cost of preservation is modest if designed in; the cost of retrofit is prohibitive.
- Multi-jurisdictional compliance (PIPEDA, Loi 25, GDPR) must hold. Anonymous structural data avoids most consent requirements; nonetheless, user-facing transparency and an opt-out are part of the design.
- Operational discipline. A learning store that gradually accumulates PII through schema drift defeats its own purpose. The design must make the no-PII guarantee structurally hard to violate, not merely culturally discouraged.

## Considered Options

- A. Anonymized structural learning events (chosen)
- B. No learning store at all
- C. Pseudonymous event logging with delayed deletion
- D. Opt-in document retention for users who consent

## Decision Outcome

Chosen option: **A. Anonymized structural learning events**, because it is the only option that captures the contextual-intelligence signal Érable's differentiation requires while remaining within the privacy posture set in ADR-007 and preserving the future option of training a domain-specific model.

The store is called `learning_events`. Each row records one structural fact about one user interaction with the product, expressed entirely in terms of closed taxonomies, bucketed identifiers, and aggregated time. No row contains PII. No row can be linked to a specific user. No row contains free text. No row contains source content.

### Sub-decision 1: Schema is built from closed taxonomies, not free text

Every field that could carry meaning is an identifier into a maintained taxonomy: `document_category` is one of a finite set (e.g. `irrc_decision_refusal`, `pay_slip_qc`, `eca_report_wes`); `interpretation_template_id` references one of the templated explanations Érable produces; `action_id` references the canonical action recommendations. Free text is forbidden in the schema. If a new category is needed, the taxonomy is extended deliberately, with a migration, never inferred at write time.

This eliminates the most common source of accidental PII leakage in event stores: text fields that capture user input or model output verbatim.

### Sub-decision 2: User profiles are bucketed, not identified

Each event references a `user_profile_bucket` rather than a user id. A bucket combines coarse demographic attributes (province, status, family composition, language, approximate time since arrival) into a label like `ON-PR-family4-FR-arrived6mo`. Buckets are coarse enough that the smallest bucket contains at least 50 users at any point; if a bucket falls below the threshold, it merges with an adjacent one until the threshold is restored.

Buckets are computed at event-write time from the user's current profile. They are not stored alongside the user record; the mapping from user to bucket is recomputed deterministically from profile attributes, which means the user can change profile attributes (and therefore bucket) without leaving a join trail.

This is the load-bearing privacy property. If buckets are coarse enough, no event can be tied back to one user even with full access to the events table.

### Sub-decision 3: Time is bucketed, not precise

Events do not record exact timestamps. They record `time_bucket`, typically the ISO week (`2026-W19`) or month, depending on the volume and sensitivity of the event class. This blocks correlation attacks where a sequence of timestamps could re-identify a user even when individual events are anonymous.

Real-time monitoring (rate limiting, anomaly detection, performance) uses a separate ephemeral metrics path that is never joined with the learning store.

### Sub-decision 4: The schema is reviewed before any addition

Adding a field to `learning_events` requires a small written justification: what signal it captures, why no closed taxonomy or bucket can carry the same signal, and why it cannot enable re-identification when joined with other fields. The justification lives in `docs/learning-events-schema-changes/` as a dated note. This is a deliberate friction; it is the structural defense against schema drift toward PII.

### Sub-decision 5: Users can opt out

The default is that learning events are recorded, because they contain no PII and are necessary for the service to improve. An explicit toggle in user settings disables learning event collection for that account going forward. The choice is visible, not buried, and the privacy policy describes precisely what the events contain.

This positions Érable correctly under PIPEDA and Loi 25 (anonymous data does not require consent, but transparency and opt-out are best practice and reduce regulatory risk) and under GDPR (the data is outside the scope of "personal data" if anonymization is solid, but offering opt-out signals good faith).

### Sub-decision 6: The store is queryable by Érable, not by external services

Learning events live in the same PostgreSQL instance as the rest of Érable's data, in a separate schema with stricter access controls. They are never exported to third-party analytics or data warehouse services. They are not embedded in third-party dashboards. If the data leaves the database at all, it leaves as aggregated reports the developer reads, never as row-level exports.

### Consequences

- Good: captures the strategic-intelligence signal — situation, document type, interpretation, recommendation, outcome — without storing source content.
- Good: privacy posture from ADR-007 holds. The events table contains no PII, no document content, no individually identifiable combinations.
- Good: future option to train a domain-specific model is preserved. After 12-24 months of operation, the events table will hold a corpus of structured (profile_bucket, document_category, interpretation, outcome) tuples that no competitor can assemble.
- Good: enables iterative service improvement immediately, before any model training. Aggregate analysis of `learning_events` reveals which interpretations get corrected most, which recommendations lead to successful outcomes, where the OCR confidence is lowest.
- Good: defensible to regulators. The data is genuinely anonymous by construction, not pseudonymous-with-promises.
- Bad: the no-PII guarantee depends on discipline. The taxonomies must be maintained, the bucket sizes must be enforced, the schema reviews must happen. A weak moment can introduce a free-text field that quietly reintroduces PII.
- Bad: bucketing loses signal. The model that could be trained on bucketed data is necessarily coarser than one trained on individual user trajectories. We accept the loss of resolution as the price of the privacy guarantee.
- Bad: outcome data ("did the recommended action succeed?") requires user follow-up that may or may not happen. Outcome is the most valuable column and the most sparsely populated. Engineering effort to collect it gracefully (without becoming intrusive) is real.
- Bad: real-time signals (rate limiting, anomaly detection) need a separate ephemeral pipeline. Two pipelines instead of one is more complexity to operate.

### Schema at a glance

The store lives in PostgreSQL under a `learning` schema, separated from operational tables. Its core table:

```sql
CREATE SCHEMA learning;

CREATE TABLE learning.events (
    event_id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    event_type            TEXT NOT NULL,         -- closed enum: 'extraction', 'interpretation', 'recommendation', 'outcome'
    user_profile_bucket   TEXT NOT NULL,         -- coarse demographic label
    document_category     TEXT,                  -- closed taxonomy id, nullable for non-document events
    interpretation_id     TEXT,                  -- template id, never the rendered text
    actions_recommended   TEXT[],                -- array of action ids
    actions_taken         TEXT[],                -- subset of recommended, when observable
    correction_signal     JSONB,                 -- {"fields_corrected": ["motif"], "n_fields_total": 12} — counts only, no content
    feedback              TEXT,                  -- closed enum: 'correct', 'wrong', 'partial', 'none'
    outcome               TEXT,                  -- closed enum: 'success', 'failure', 'unknown', null until reported
    time_bucket           TEXT NOT NULL          -- ISO week, e.g. '2026-W19'
);

CREATE INDEX events_bucket_category ON learning.events (user_profile_bucket, document_category);
CREATE INDEX events_time_bucket ON learning.events (time_bucket);
```

Note what is absent: no user id, no foreign key to the users table, no email hash, no IP address, no precise timestamp, no free-text column. The schema's negative space is the privacy guarantee.

A profile bucket helper enforces the minimum-size rule:

```python
def compute_profile_bucket(profile: UserProfile) -> str:
    raw_bucket = f"{profile.province}-{profile.status}-family{profile.family_size}-{profile.language}-arrived{profile.arrival_quarter}"
    if bucket_size(raw_bucket) < MIN_BUCKET_SIZE:
        return coalesce_to_parent_bucket(raw_bucket)
    return raw_bucket
```

The coalescing function moves users into broader buckets when their granular bucket would be too small to hide them.

## Pros and Cons of the Options

### Option A: Anonymized structural learning events

- Good: captures the contextual-intelligence signal that differentiates Érable.
- Good: no PII, by construction. Compatible with ADR-007 and verifiable in code.
- Good: preserves the future option of training a domain-specific model.
- Good: enables service improvement on day one, well before any model training.
- Bad: requires ongoing discipline (taxonomies, bucket sizes, schema reviews).
- Bad: outcome column is sparse without thoughtful UX to collect follow-up.
- Bad: bucketing reduces resolution compared to individual trajectories.

### Option B: No learning store at all

- Good: simplest possible privacy story. Nothing to leak, nothing to defend.
- Good: lowest implementation cost.
- Bad: forecloses the strategic differentiation. Érable becomes a wrapper around generic models with no path to deepen its understanding of Canadian administrative reality.
- Bad: forecloses the future option to train a domain-specific model. Even modest improvements (better prompts, better recommendations) become guesswork rather than data-informed.
- Bad: the privacy claim is preserved but the product weakens over time relative to competitors who do learn from usage, however imperfectly.
- Neutral: a defensible choice for a product whose value sits entirely in immediate utility, with no learning curve to climb. Not Érable's shape.

### Option C: Pseudonymous event logging with delayed deletion

- Good: keeps individual user trajectories intact, which yields a richer signal for some learning tasks (longitudinal analysis, churn prediction).
- Good: better signal density than bucketing.
- Bad: pseudonymous identifiers are notoriously easy to re-identify when combined with other fields. The "anonymous" claim becomes brittle and difficult to defend in audit.
- Bad: requires deletion schedules, retention policies, and data subject access workflows. The operational cost is real.
- Bad: weakens the verifiable privacy story. Where ADR-007 says "we don't store this", a pseudonymous log says "we store it differently". The marketing and trust posture suffers.
- Neutral: industry-standard practice in many SaaS products. Not where Érable wants to position itself.

### Option D: Opt-in document retention for users who consent

- Good: highest signal possible — full document content from consenting users.
- Good: legally clean if consent is explicit, granular, and revocable.
- Bad: contradicts the simple "we never store your documents" claim. The marketing story splinters into "we don't store them, unless you let us".
- Bad: opt-in consent rates for this kind of donation are historically low (5-15%), so the dataset is small and self-selected.
- Bad: the consenting subset is not representative — users who consent are more privacy-comfortable than the general population, biasing any model trained on the data.
- Bad: re-identification risk is high. Even anonymized, real document content is hard to scrub completely.
- Neutral: a viable last resort if option A produces insufficient signal after several years. Not a starting point.

## More Information

- Related ADRs:
  - ADR-007 (privacy-radical architecture) — sets the posture this ADR operationalizes. ADR-007 says what we do not store; this ADR says what we do, and how the difference is defended.
  - ADR-006 (pgvector over dedicated vector DB) — the same PostgreSQL instance hosts both the application data and the learning store, simplifying operations.
  - Future ADR on RAG architecture — interpretation templates referenced by `interpretation_id` are produced by the RAG pipeline; their identifiers are stable and versioned.
- References:
  - PIPEDA guidance on anonymization and de-identification.
  - Loi 25 (Québec) provisions on de-identified information (Section 23).
  - GDPR Recital 26 on truly anonymous data falling outside the regulation's scope.
  - Sweeney, L. (2002), "k-anonymity: a model for protecting privacy", as background on bucketing thresholds.
- Revisit trigger: revisit this decision if any of the following occur:
  - Schema reviews (Sub-decision 4) start being skipped, suggesting the structural defense against PII drift is eroding.
  - Bucket sizes consistently fall below the minimum despite coalescing rules, indicating that the demographic dimensions are too narrow for the user base.
  - The accumulated dataset, after 18 to 24 months, proves insufficient to drive meaningful service improvement, requiring a deliberate reopening of the trade-offs in ADR-007.
  - A specific learning task emerges that genuinely cannot be served by structural events and where source content is necessary; that task is evaluated against ADR-007's revisit trigger, not this one.
  - Regulatory or jurisprudential developments raise the bar on what counts as anonymous, requiring the bucketing or coalescing rules to be tightened.
