# ADR-001: FastAPI with ARQ over Django with Celery

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable is a greenfield Python backend serving a bilingual web app for francophone newcomers to Canada. The product surface combines synchronous CRUD (user accounts, onboarding, dashboards), latency-sensitive streaming (LLM chat responses over SSE), and a background workload that grows over time (weekly re-crawl of official Canadian sources, embedding generation, OCR pipelines, scheduled email reminders).

Two coupled choices are needed before the first endpoint ships: the web framework and the background job runner. They are coupled because Django and FastAPI each push toward a different async model, a different ORM style, and a different idiomatic queue. Mixing them is possible but rarely clean.

The decision is needed now because every later choice — SQLAlchemy version, Pydantic usage, auth flow, deployment topology — depends on it. Postponing means rewriting bootstrap code in week 3.

## Decision Drivers

- Async-native streaming is a core feature: LLM chat over SSE, multi-second responses, hundreds of concurrent open connections expected at modest user counts.
- Background jobs are first-class, not an afterthought: weekly source re-crawl, embedding refresh, async OCR, scheduled emails. The worker must be reliable enough to run unattended for weeks.
- Career signal for Ottawa hiring (priority 1 per project rules): the chosen stack must be one a recruiter recognizes immediately and one I can defend in interviews.
- Single-developer velocity for 10-12 weeks: the framework must not require a steep ramp-up before the first useful endpoint.
- Stack coherence: avoid mixing asyncio code paths with thread-pool workarounds. One concurrency model, end to end.
- Type safety and OpenAPI generation out of the box, since the frontend is TypeScript strict and benefits from generated clients.

## Considered Options

- A. FastAPI + ARQ
- B. Django + Celery
- C. Litestar + ARQ
- D. FastAPI + Celery

## Decision Outcome

Chosen option: **A. FastAPI + ARQ**, because it is the only combination that is async end to end, aligns with the Pydantic v2 / SQLAlchemy 2.0 async ecosystem we already need for RAG, and is the dominant signal in modern Python job postings while remaining defensible in an interview.

### Consequences

- Good: one concurrency model across web and workers; no thread-pool bridges, no `sync_to_async` wrappers.
- Good: native SSE and WebSocket support without extra middleware, which the LLM chat needs from week 9.
- Good: Pydantic v2 schemas are reused for request validation, OpenAPI generation, and LLM structured outputs (see future ADR on LLM gateway).
- Good: ARQ is small enough to read end to end in an afternoon, which makes debugging stuck jobs tractable for a solo developer.
- Bad: less batteries-included than Django. We write our own admin views, our own auth flow (already accepted in D22 / future ADR-004), and assemble migrations, settings, and project layout by hand.
- Bad: ARQ has a smaller community than Celery. Fewer Stack Overflow answers, fewer plugins, no Flower-equivalent UI out of the box.
- Bad: weaker signal for parts of the Ottawa market — federal contractors and older enterprise shops still run Django. We accept this trade-off because the modern startup and scale-up segment we target leans FastAPI.
- Bad: long-running jobs (>30 min wall time) are not ARQ's sweet spot. If we hit that ceiling we will revisit (see Revisit trigger).

## Pros and Cons of the Options

### Option A: FastAPI + ARQ

- Good: async from request to worker; matches the SSE streaming and concurrent LLM-call workload directly.
- Good: Pydantic v2 is a first-class citizen, not a bolt-on. Validation, settings, and OpenAPI all share the same schema definitions.
- Good: ARQ uses Redis, which we already need for caching and rate limiting. No extra infrastructure.
- Good: high signal in current Python job postings, especially in AI/LLM-adjacent roles.
- Bad: no admin UI, no built-in auth, no ORM. Everything is assembled.
- Bad: ARQ's ecosystem is small; we own more of the operational surface (monitoring, retries, dead-letter handling).

### Option B: Django + Celery

- Good: most mature Python web stack. Admin, ORM, migrations, auth, forms, sessions all included.
- Good: Celery is the de facto job queue; massive ecosystem, Flower for monitoring, well-known operational patterns.
- Good: strong signal in Canadian government and established enterprise Ottawa shops.
- Bad: async story is still bolted on. Django ASGI works, but most of the ORM and middleware ecosystem assumes sync. SSE and long-lived connections require care.
- Bad: Celery with asyncio is awkward — you end up running async code inside sync workers via `asyncio.run`, which negates the model.
- Bad: heavy framework for a single-developer 12-week MVP where most Django features (admin, forms, templating) go unused because the frontend is Next.js.
- Neutral: Django REST Framework is excellent but adds another layer between us and the request.

### Option C: Litestar + ARQ

- Good: async-native, modern, type-driven. Arguably cleaner architecture than FastAPI (better dependency injection, plugin system).
- Good: pairs naturally with ARQ for the same reasons FastAPI does.
- Bad: smaller community than FastAPI by an order of magnitude. Fewer tutorials, fewer integrations, less coverage in LLM tooling docs (Anthropic, LiteLLM, Langfuse all default to FastAPI examples).
- Bad: weaker recruiter recognition. A hiring manager scanning a CV recognizes FastAPI immediately; Litestar invites a "what's that?" instead of moving the conversation forward.
- Neutral: technically defensible but loses on the career-signal driver, which is priority 1.

### Option D: FastAPI + Celery

- Good: keeps FastAPI's async web layer while leaning on Celery's mature worker ecosystem.
- Good: Celery's scheduling, retries, and monitoring are battle-tested.
- Bad: the boundary between async FastAPI and sync Celery is exactly the seam that produces hard-to-debug bugs. Workers run sync, but our code base is async. Calling async LLM clients from a sync worker means `asyncio.run` per task, which is wasteful and easy to get wrong.
- Bad: two concurrency models in one codebase. A solo developer should not pay this cost without a strong reason.
- Neutral: a sensible choice for a team migrating from Django, but we have no migration to honor.

## More Information

- Related ADRs: ADR-004 (auth flow built from scratch), ADR-005 (LiteLLM gateway), ADR-006 (pgvector over dedicated vector DB). All assume an async Python stack.
- References:
  - FastAPI documentation, async background tasks and dependency injection sections.
  - ARQ documentation, particularly the cron and retry semantics.
  - Pydantic v2 release notes; performance and validator changes that motivate v2 over v1.
- Revisit trigger: revisit this decision if any of the following occur:
  - We need batch jobs with wall time exceeding 30 minutes (ARQ is not designed for this; Celery or a workflow engine like Temporal becomes warranted).
  - We hire a developer whose primary expertise is Django and onboarding cost outweighs framework lock-in.
  - FastAPI 1.0 introduces a breaking migration that materially raises maintenance cost.
