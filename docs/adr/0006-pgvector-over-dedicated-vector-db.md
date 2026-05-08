# ADR-006: pgvector over a dedicated vector database

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable's core value relies on retrieval-augmented generation over Canadian official sources: federal pages on canada.ca, provincial pages on ontario.ca and quebec.ca, plus organism websites like OCISO and CESOC. Users ask questions in French or English, and the system must surface passages whose meaning matches the question, even when the wording, the language, or the terminology differ. This is semantic search, and it is implemented by storing dense embeddings of text chunks and querying them by similarity.

A storage and indexing layer for these embeddings must be chosen before any RAG code lands in week 7. The decision constrains how chunks are written, how queries are issued, how data is backed up, and how the operational topology looks.

Two architectural shapes are possible. The first stores vectors in a dedicated vector database, separate from the relational data. The second uses the existing PostgreSQL instance with the pgvector extension, keeping all data in one engine.

## Decision Drivers

- Operational simplicity for a solo developer. Each additional service to deploy, monitor, back up, and upgrade is an ongoing cost.
- Transactional and query coherence between vector data and relational metadata. Chunks have attributes (source URL, last-verified date, province, language) that drive both filtering and freshness logic.
- Realistic volume for the foreseeable horizon. Initial corpus is on the order of a few thousand chunks; full Ottawa coverage lands at low tens of thousands; multi-province expansion at low hundreds of thousands. Comfortably within pgvector's well-documented operating range.
- Zero additional fixed cost during pre-revenue. Managed vector services start at meaningful monthly fees regardless of usage.
- No career signal driver. The choice of vector storage engine is not a CV-relevant differentiator. Recognized RAG competence sits in chunking strategy, retrieval quality, reranking, and evaluation — all layered above the storage engine.

## Considered Options

- A. pgvector (PostgreSQL extension)
- B. Pinecone (managed proprietary)
- C. Qdrant (open source, self-hosted or managed)
- D. Weaviate (open source, self-hosted or managed)
- E. Chroma (open source, lightweight)

## Decision Outcome

Chosen option: **A. pgvector**, because it eliminates an entire service from the operational footprint, lets us join vector results with relational metadata in one query, and is more than sufficient for the realistic volume horizon. The convenience that a dedicated vector database offers does not justify a second database for our scale and team size.

### Consequences

- Good: one database to operate, back up, restore, monitor, secure, and upgrade. No extra service in docker-compose, no extra credentials, no extra failure mode.
- Good: vector similarity and relational filtering compose naturally in SQL. "Top 10 chunks by similarity, where source is ontario.ca and last_verified within 14 days" is one query, not two with a join in application code.
- Good: ACID guarantees across vector and relational data. Re-embedding a source after re-crawl updates vectors and freshness metadata in one transaction.
- Good: zero additional fixed cost. The PostgreSQL instance is already provisioned for the rest of the application.
- Good: backup story is unified. Existing pg_dump or physical backups cover vectors automatically.
- Bad: filtered approximate nearest neighbor search (combining ANN with rich predicates) is less efficient in pgvector than in Qdrant, which supports payload filters natively at the index level. At our volume this is imperceptible; at 10M+ vectors with complex filters it would matter.
- Bad: HNSW index tuning (parameters m, ef_construction, ef_search) requires understanding what the parameters do. A managed vector database hides this behind a UI. We accept the tuning cost as part of operating PostgreSQL.
- Bad: no native multi-tenant isolation at the index level. If the B2B white-label tier ever requires hard tenant separation in vector search, the design will need a deliberate sharding scheme or a different engine for that tier.

### Schema and index choices at a glance

The vector storage lives in a single `chunks` table that joins relational metadata and embeddings. The schema is intentionally narrow:

```sql
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE chunks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_url      TEXT NOT NULL,
    source_domain   TEXT NOT NULL,            -- e.g. 'ontario.ca'
    language        TEXT NOT NULL,            -- 'fr' or 'en'
    province        TEXT,                     -- 'ON', 'QC', null for federal
    content         TEXT NOT NULL,
    content_hash    TEXT NOT NULL,            -- for diff detection on re-crawl
    embedding       vector(1024) NOT NULL,    -- BGE-M3 produces 1024-dim vectors
    last_verified_at TIMESTAMPTZ NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX chunks_embedding_hnsw
    ON chunks
    USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

CREATE INDEX chunks_source_domain ON chunks (source_domain);
CREATE INDEX chunks_last_verified_at ON chunks (last_verified_at);
```

Concrete choices and the reason for each:

- **HNSW over IVFFlat**: better recall at comparable build time, and HNSW does not require periodic rebuilds when data grows. IVFFlat would force us to rebuild the index every time the corpus grows materially.
- **Cosine distance (`<=>`) over L2 or inner product**: BGE-M3 produces normalized embeddings, and cosine is the conventional metric for normalized text embeddings.
- **1024 dimensions**: dictated by BGE-M3, our chosen embedding model.
- **`m = 16, ef_construction = 64`**: pgvector's documented defaults, suitable for corpora up to several million vectors. Tunable later based on measured recall and latency.

Queries combine similarity and predicates in one SQL statement:

```sql
SELECT id, content, source_url, last_verified_at,
       embedding <=> $1 AS distance
FROM chunks
WHERE source_domain IN ('ontario.ca', 'canada.ca')
  AND last_verified_at > now() - interval '30 days'
ORDER BY embedding <=> $1
LIMIT 10;
```

## Pros and Cons of the Options

### Option A: pgvector

- Good: single database. One operational surface, one backup, one set of credentials.
- Good: SQL-native joins between vector results and relational metadata. Filtering and similarity compose in one query.
- Good: ACID transactions across vector and relational data. Re-embedding is consistent with metadata updates by construction.
- Good: zero additional cost. PostgreSQL is already in the stack.
- Good: HNSW index supports the volumes we project, with documented behavior at single-digit-million vector scale.
- Bad: filtered ANN search is less efficient than dedicated engines on complex predicates at large volumes.
- Bad: index tuning is the operator's responsibility.
- Neutral: the pgvector extension is mature, widely used, and continues to gain features (binary quantization, halfvec, etc.).

### Option B: Pinecone

- Good: fully managed, no operational burden on the developer.
- Good: scales horizontally without intervention.
- Good: best-in-class for billion-vector workloads with high QPS.
- Bad: closed source, vendor lock-in. Migration off Pinecone is a real project.
- Bad: separate billing layer that scales with usage. Free tier is limited; serious use starts at meaningful monthly cost.
- Bad: data leaves our infrastructure, which conflicts with the privacy-first posture in PRIVACY.md and PIPEDA/Loi 25 alignment.
- Bad: vector data lives in a different system from relational metadata, requiring application-level joins and a separate consistency story for re-crawls.
- Neutral: a credible choice for a well-funded project at scale. Disproportionate for our shape.

### Option C: Qdrant

- Good: open source, self-hostable. No vendor lock-in.
- Good: excellent filtered ANN search, payload filters integrated at the index level.
- Good: written in Rust, low resource footprint, fast.
- Good: can be operated as a separate service or via managed cloud.
- Bad: still a separate service to deploy, monitor, back up, and upgrade.
- Bad: separate consistency story between vector data and relational metadata.
- Bad: marginal benefit at our scale; the strengths (advanced filtering, very high volumes) do not pay off until we are well past MVP scope.
- Neutral: the most technically attractive alternative if we ever outgrow pgvector. Worth revisiting at the threshold described below.

### Option D: Weaviate

- Good: open source, schema-driven, supports modules for embedding generation, hybrid search, and reranking.
- Good: rich feature surface, including built-in BM25 and hybrid search modes.
- Bad: heavier than Qdrant in memory and operational complexity.
- Bad: schema model is opinionated; integrating it with our relational schema duplicates concepts.
- Bad: same separate-service trade-offs as Qdrant.
- Neutral: a strong product, but its strengths target use cases more sophisticated than ours.

### Option E: Chroma

- Good: lightweight, easy to embed in a Python process for prototyping.
- Good: minimal operational overhead in toy setups.
- Bad: production story for persistence, scale, and operations is significantly less mature than the alternatives.
- Bad: not designed for the multi-environment, multi-developer-team trajectory Érable is on.
- Neutral: a reasonable choice for a notebook or a hackathon. Not a serious candidate for a production system.

## More Information

- Related ADRs:
  - ADR-001 (FastAPI + ARQ) — async PostgreSQL access through SQLAlchemy 2.0 covers vector queries the same way as relational ones.
  - Future ADR on RAG architecture (planned for S4) — defines chunking strategy, retrieval pipeline, and reranking approach. This ADR provides the storage substrate; that ADR will provide the retrieval logic.
  - Future ADR on freshness pipeline — re-crawl, diff, and re-embedding strategy, which exploits the transactional coherence chosen here.
- Excluded from detailed analysis:
  - **pg_embedding**: the alternative PostgreSQL vector extension launched by Neon in 2023. Effectively abandoned by late 2024, with Neon itself recommending pgvector. Mentioned only to confirm it was considered.
  - **Milvus**: enterprise-oriented, operational complexity disproportionate to project scope.
  - **Elasticsearch with dense vectors**: viable but overkill, and the Java footprint is at odds with our otherwise-Python infrastructure.
- References:
  - pgvector documentation, particularly the HNSW and IVFFlat index sections.
  - BGE-M3 model card and embedding dimensions.
  - PostgreSQL 16 documentation on extensions and index tuning.
- Revisit trigger: revisit this decision if any of the following occur:
  - Corpus exceeds ~5 million chunks AND sustained query load exceeds ~50 vector queries per second, at which point pgvector tuning costs may exceed the operational cost of a dedicated engine.
  - Filtered ANN search on complex predicates becomes a measured bottleneck (latency budget exceeded by predicate-heavy queries).
  - The B2B white-label tier requires hard multi-tenant isolation at the vector level that is impractical to express within a shared PostgreSQL instance.
  - Re-embedding cycles begin to interfere with online query latency in measurable ways, suggesting workload separation between transactional and analytical paths.
