# Erable

> Your guide to settle in Canada — without stress, without mistakes.

Erable is an AI-powered web app that helps French-speaking newcomers
navigate the administrative steps of settling in Canada: getting a SIN,
opening a bank account, applying for OHIP, finding childcare, filing
their first tax return, and a dozen other things that usually take
weeks to figure out alone.

The first phase focuses on Ottawa, Ontario. More provinces and cities
will follow.

A French version of this README is available: [README.fr.md](./README.fr.md).

---

## Status

**In development. Public launch planned ~12 weeks from kickoff.**

This repository is public from day one. The codebase is the product —
there is no closed-source enterprise edition, no premium-only modules
hidden away. The business model relies on transparent affiliate
partnerships, one-shot AI services, and a Premium subscription, all of
which are documented openly.

---

## Why this exists

The author is himself a French-speaking newcomer arriving in Ottawa.
Every pain point Erable addresses is a pain point he is currently
experiencing or about to experience.

Existing resources are either outdated (Settlement.org), English-only
(WelcomeAide, Moving2Canada), bank-biased (RBC's Arrive app), or
expensive (RCIC consultants at $500–$1,500). Erable aims to fill the
gap with modern UX, AI assistance, and a francophone-first approach,
while staying strictly within the legal scope of public information
(no immigration advice — that is reserved for licensed RCICs).

---

## Tech stack

**Backend** — Python 3.13, FastAPI, SQLAlchemy 2.0 async, PostgreSQL 16
with pgvector, Redis 7, ARQ for background workers, Pydantic v2.

**Frontend** — Next.js 14 (App Router), TypeScript strict, Tailwind CSS,
shadcn/ui, next-intl for FR/EN i18n, PWA.

**AI** — Provider-agnostic LLM abstraction supporting Anthropic Claude,
OpenAI, Mistral, and local Ollama. BGE-M3 embeddings for multilingual
RAG. Tesseract for privacy-first local OCR; Claude Vision only as an
opt-in fallback.

**Infrastructure** — Docker, Hetzner VPS with Coolify for production,
Cloudflare for edge, GitHub Actions for CI/CD. AWS Terraform templates
are kept in `infrastructure/aws/` as a reference, not as the production
target.

**Observability** — structlog, OpenTelemetry, Sentry, Grafana Cloud.

Architecture decisions are tracked in [`docs/adr/`](./docs/adr).

---

## Privacy

Privacy is not a feature, it is a position. Documents uploaded for
analysis are processed in memory and never persisted to disk or
database. The OCR pipeline runs locally with Tesseract by default;
sending a document to Claude Vision requires an explicit per-document
consent. Affiliate tracking applies to authenticated users only — no
anonymous fingerprinting, no third-party cookies.

The full privacy policy lives in [PRIVACY.md](./PRIVACY.md) and is
written in plain language, not legalese.

---

## Legal scope

Erable provides general information based on official Canadian public
sources. **Erable is not an immigration, legal, or tax advisory
service.** For personalized advice, consult a Regulated Canadian
Immigration Consultant (RCIC) or a lawyer admitted to the bar of the
relevant province.

This disclaimer is enforced in the product itself: the AI is tested to
refuse questions that fall outside the scope of public information,
and these tests are part of the CI pipeline.

---

## Quickstart

> The bootstrap is in progress. Detailed setup instructions will land
> with the first runnable version of the stack. For now, this section
> documents the intended developer experience.

**Prerequisites**

- Docker 25+ and Docker Compose v2
- Node.js 20+ and pnpm 9+
- Python 3.13+ and [uv](https://github.com/astral-sh/uv)
- GNU Make
- A Linux or macOS host (Fedora 44 is the reference development environment)

**Setup**

```sh
git clone git@github.com:mdsiha/erable.git
cd erable
cp .env.example .env  # then fill in local secrets
make dev              # starts postgres, redis, backend, frontend
```

A `Makefile` at the repository root will be the single entry point
for common tasks (`make dev`, `make test`, `make lint`, `make migrate`).

---

## Repository layout

```
erable/
├── backend/          FastAPI service + ARQ workers
├── frontend/         Next.js application
├── infrastructure/   Coolify configs (prod) and AWS Terraform (reference)
├── docs/             Architecture, ADRs, runbooks
└── .github/          CI workflows, issue and PR templates
```

---

## Contributing

The project is in heavy active development by a single author during
the first 10–12 weeks. External contributions are welcome but may take
some time to review until the MVP is shipped.

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the contribution workflow,
and [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) for community
expectations.

To report a security issue, please follow [SECURITY.md](./SECURITY.md).

---

## Roadmap

A high-level roadmap is maintained in [ROADMAP.md](./ROADMAP.md).
Detailed work happens through GitHub Issues and Projects.

---

## License

[MIT](./LICENSE) © 2026 Michael Das

---

## Contact

General inquiries: `hello@erable.app`

For everything else, GitHub Issues are the preferred channel.
