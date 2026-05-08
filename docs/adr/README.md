# Architecture Decision Records

This directory contains the architectural decision records (ADRs) for Érable. Each ADR documents a single significant technical or strategic decision, the alternatives considered, and the trade-offs accepted.

ADRs follow the [MADR 4.0](https://adr.github.io/madr/) format with one local addition: every ADR includes a "Revisit trigger" subsection that names the conditions under which the decision should be reconsidered.

## Status legend

- **Accepted** — the decision is in force.
- **Proposed** — under discussion, not yet acted on.
- **Superseded by ADR-NNN** — replaced by a later decision.
- **Deprecated** — no longer relevant, kept for historical context.

## Index

| # | Title | Status | Summary |
|---|---|---|---|
| [001](./0001-fastapi-arq-over-django-celery.md) | FastAPI with ARQ over Django with Celery | Accepted | Async-native Python stack for the web layer and background workers, chosen for streaming LLM responses, ecosystem alignment, and modern hiring signal. |
| [002](./0002-lemon-squeezy-over-stripe-direct.md) | Lemon Squeezy as Merchant of Record over Stripe direct | Accepted | MoR platform handles cross-border tax compliance and works from Turkey today, with a future migration path to Stripe Managed Payments after Canadian incorporation. |
| [003](./0003-hetzner-coolify-over-aws-production.md) | Hetzner VPS with Coolify over AWS for production hosting | Accepted | Right-sized infrastructure for solo MVP scale; AWS career signal is built deliberately on a separate side project rather than carried by production. |
| [004](./0004-auth-from-scratch.md) | Build authentication from scratch on standard primitives | Accepted | Custom JWT plus refresh-rotation-with-reuse-detection, magic-link login, and httpOnly cookies, on argon2id and signed JWTs, with documented threat model and external review before production. |
| [005](./0005-litellm-with-facade.md) | LiteLLM with a thin in-house facade for LLM access | Accepted | LiteLLM in library mode for Ollama-dev / Anthropic-prod symmetry, wrapped by a minimal Pydantic-typed facade that keeps the gateway swappable. |
| [006](./0006-pgvector-over-dedicated-vector-db.md) | pgvector over a dedicated vector database | Accepted | Single PostgreSQL instance for relational and vector data, with HNSW indexing on 1024-dim BGE-M3 embeddings, sufficient for the projected corpus volume. |
| [007](./0007-privacy-radical-architecture.md) | Privacy-radical architecture | Accepted | No persistent storage of user-submitted documents, local-first OCR with explicit opt-in for cloud, PII out of logs, authenticated-only affiliate tracking, 60-second deletion contract enforced at three layers. |
| [008](./0008-privacy-preserving-learning-architecture.md) | Privacy-preserving learning architecture | Accepted | Anonymized `learning_events` store built on closed taxonomies, bucketed user profiles, and bucketed time, capturing service-improvement signal and preserving the future option of a domain-specific model without weakening ADR-007. |

## How to read these ADRs

Each ADR is self-contained and answers four questions:

1. What forces are at play (Context).
2. What was considered (Considered Options).
3. What was chosen and why (Decision Outcome).
4. When to reconsider (Revisit trigger, in More Information).

ADRs are not policy documents. They record decisions made at a specific moment with the information available then. When circumstances change, a new ADR supersedes the old one rather than editing it. The history matters as much as the current state.

## Adding a new ADR

1. Pick the next number in sequence (`NNNN-kebab-case-title.md`).
2. Use MADR 4.0 format. The existing ADRs are working templates.
3. Status starts as `Proposed`. Move to `Accepted` once the decision is acted on (typically: code merged that depends on it).
4. Update this index.
5. Commit with a conventional commit: `docs(adr): add ADR-NNN on <topic>`.

If a new ADR replaces an existing one, mark the old ADR as `Superseded by ADR-NNN` and add a forward reference in its first section. Do not delete superseded ADRs.
