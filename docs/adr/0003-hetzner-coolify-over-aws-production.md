# ADR-003: Hetzner VPS with Coolify over AWS for production hosting

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable needs a production hosting target before week 10, when the first public deployment ships at erable.app. The MVP serves a Next.js frontend, a FastAPI backend, an ARQ worker pool, PostgreSQL with pgvector, Redis, and a MinIO instance for file storage. Realistic load for the first six months is 100 to 1000 monthly active users, with a long tail of background work (weekly source re-crawl, embedding refresh, OCR pipelines).

The hosting decision sits at an unusual intersection of forces. On the operational side, the project is run solo, with no dedicated ops time, and a Hetzner VPS is already paid for. On the career side, Ottawa hiring favors AWS literacy, and Érable is explicitly an instrument for the founder's career strategy (priority 1 in project rules).

These two forces seem to point in opposite directions. The decision must address both without compromising either: ship a real MVP fast, and build credible cloud credentials for the Ottawa job market.

## Decision Drivers

- Time to first production deployment with one developer and zero dedicated ops hours.
- Total cost of ownership during the pre-revenue phase, where every dollar of fixed infrastructure cost is a dollar not spent on tools, education, or marketing.
- Career signal for Ottawa hiring, where AWS literacy is a near-default expectation. This driver is real but addressed separately via D26 (a distinct AWS side project planned for months 6-8) rather than by the production stack.
- Right-sizing for the actual load profile of an MVP. 100 to 1000 monthly users do not need multi-AZ failover, autoscaling groups, or edge compute.
- Lock-in posture. Open infrastructure that can be migrated later beats proprietary infrastructure that locks the architecture in early.

## Considered Options

- A. Hetzner VPS + Coolify
- B. AWS production stack (ECS Fargate, RDS, ElastiCache, ALB, Route 53, CloudFront)
- C. Hetzner VPS + Docker Compose + manual reverse proxy
- D. Fly.io
- E. Managed PaaS (Railway, Render)

## Decision Outcome

Chosen option: **A. Hetzner VPS + Coolify**, because it minimizes time to first deploy, reuses an already-paid resource, and lets the AWS career signal be built deliberately on a separate side project where the cloud-native patterns are the point. The career driver is satisfied without forcing the MVP to carry it.

### Consequences

- Good: production-ready in days, not weeks. Coolify handles reverse proxy, automatic SSL via Let's Encrypt, container deployments, database backups, and basic monitoring out of the box.
- Good: fixed monthly cost, predictable. No surprise bills from a misconfigured Lambda or a runaway egress charge.
- Good: leaves the AWS learning surface for the dedicated side project (D26) where Lambda, DynamoDB, EventBridge, Terraform, and CloudWatch can be exercised cleanly without entangling Érable's production reliability.
- Good: choosing right-sized infrastructure for the project stage is itself a signal of operational judgment, not a lack of cloud knowledge. This is defensible in interviews when paired with the AWS side project.
- Good: Coolify is open source, and the underlying stack (Docker, Traefik, Postgres, Redis) is fully portable. Migration to AWS, Fly, or another VPS later is a matter of reapplying compose files and DNS.
- Bad: weaker raw signal for some Ottawa recruiters who scan stacks for AWS keywords. We accept this trade-off because D26 addresses the gap, and because a working product on Hetzner is a stronger signal than half-finished AWS infrastructure.
- Bad: single-VPS topology has no built-in high availability. Hardware failure or a Hetzner network incident takes the site down. Acceptable for MVP load; revisit when SLAs are needed.
- Bad: Coolify is younger than the alternatives. Bugs and rough edges exist. We accept this in exchange for the speed it gives us today.

## Pros and Cons of the Options

### Option A: Hetzner VPS + Coolify

- Good: dashboard-driven deploys, automatic SSL, integrated Postgres and Redis provisioning, GitHub integration. A solo developer can ship in an afternoon.
- Good: Hetzner pricing is among the lowest in the European market for the resources offered (CPU, RAM, NVMe, bandwidth).
- Good: Coolify is open source and self-hosted; no platform lock-in.
- Good: the underlying stack is industry-standard (Docker, Traefik), so skills transfer.
- Bad: single point of failure unless we provision multiple VPSes and orchestrate them ourselves, at which point the simplicity advantage disappears.
- Bad: Coolify's ecosystem is smaller than AWS's. Less Stack Overflow coverage when something breaks.
- Neutral: weaker recruiter recognition than AWS, addressed by D26.

### Option B: AWS production stack

- Good: industry standard, recognized everywhere, deepest hiring signal in Ottawa.
- Good: every component (compute, DB, cache, queue, storage, CDN, DNS) has a managed AWS equivalent. The mental model scales from MVP to enterprise.
- Good: best-in-class observability, IAM, and security primitives.
- Bad: highest operational complexity. ECS task definitions, IAM policies, VPC configuration, security groups, and CloudFormation or Terraform are non-trivial to set up correctly for one person under MVP time pressure.
- Bad: highest cost during pre-revenue. Even modest AWS setups (RDS db.t4g.small, single-AZ, 1 ECS service, ALB, basic CloudWatch) clear 80-150 USD per month before traffic. Cost scales with mistakes.
- Bad: easy to misconfigure in ways that create bills or security issues, neither of which a solo founder should be debugging at week 10.
- Neutral: defensible if the founder were already AWS-fluent, which is not yet the case (see lacuna in PROFILE_AND_LEARNING.md).

### Option C: Hetzner VPS + Docker Compose + manual reverse proxy

- Good: maximum control. No layer between the developer and the OS.
- Good: lower abstraction tax than Coolify. Useful as a learning exercise.
- Bad: every concern Coolify abstracts away (SSL, deploys, backups, basic dashboards, GitHub integration) becomes a script to write and maintain.
- Bad: time spent on this is time not spent on RAG, content, or career artifacts. For a 12-week MVP solo, the cost is too high.
- Neutral: a reasonable choice for a developer with strong existing infra muscle memory who wants to minimize dependencies. Not the right trade-off here.

### Option D: Fly.io

- Good: modern PaaS with declarative deploys, regional placement, native Postgres, edge-friendly.
- Good: async-friendly architecture aligns with FastAPI + ARQ.
- Bad: pricing has been volatile; recent changes have made it less attractive for resource-heavy workloads.
- Bad: still a single-vendor dependency, with smaller ecosystem and less recruiter recognition than AWS.
- Bad: does not reuse the existing paid Hetzner VPS.
- Neutral: technically credible alternative; loses on cost and on the career argument addressed elsewhere.

### Option E: Managed PaaS (Railway, Render)

- Good: simplest possible developer experience. Deploy from Git, click to add Postgres, done.
- Good: very low time-to-first-deploy, comparable to Coolify.
- Bad: more expensive than Hetzner at equivalent resource levels, especially for the worker pool.
- Bad: stronger lock-in to the platform's deployment model than Coolify; migration is a real effort if pricing changes.
- Bad: no career signal advantage over Hetzner + Coolify, and arguably weaker.
- Neutral: a fine choice for a founder with no existing VPS, which is not our situation.

## More Information

- Related ADRs: none directly. ADR-001 (FastAPI + ARQ) and the assumed Postgres + Redis stack constrain the hosting target to anything Docker-compatible, which all five options satisfy.
- References:
  - Hetzner Cloud pricing and VPS specifications (CCX line).
  - Coolify documentation and self-hosting guide.
  - AWS pricing calculator for an equivalent ECS Fargate + RDS + ElastiCache + ALB stack at MVP scale.
  - D26 in PROJECT.md running log (distinct AWS side project, months 6-8).
- Revisit trigger: revisit this decision if any of the following occur:
  - Sustained traffic exceeds what a Hetzner CCX-class VPS serves comfortably (rough threshold: 10k requests per minute sustained, or working set exceeding RAM, or database larger than 50 GB).
  - SLA requirements appear (B2B contracts, regulatory commitments) that demand multi-AZ or multi-region failover.
  - Edge compute or geographically distributed CDN logic is needed beyond what Cloudflare's free tier provides.
  - A new hire whose primary stack is AWS joins the team and onboarding cost outweighs migration cost.
  - The AWS side project (D26) reaches a maturity that justifies migrating Érable production into the same account for unified billing and operations.
