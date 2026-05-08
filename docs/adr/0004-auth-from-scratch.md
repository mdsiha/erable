# ADR-004: Build authentication from scratch on standard primitives

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable needs an authentication system that handles signup, login, magic-link passwordless flows, session refresh, multi-device support, and eventual SSO for the B2B white-label tier. The system must work bilingually (French and English) from day one, store no more user data than strictly necessary (PIPEDA, Loi 25, GDPR), and produce a session model that is safe against the common attack classes: token theft, replay, CSRF, XSS exfiltration, and timing attacks on credential checks.

The decision is needed in week 4, when the first authenticated endpoints ship. It is one of the highest-stakes technical choices of the project for two reasons. First, authentication mistakes are security incidents, not bugs. Second, authentication is the technical area most likely to be probed in detail during senior engineering interviews in Ottawa, which makes it a deliberate career-signal opportunity (priority 1 in project rules).

The decision covers both the backend (FastAPI: token issuance, refresh rotation, magic-link generation, password hashing) and the frontend (Next.js: token storage, automatic refresh, route protection). They are coupled and decided together.

## Decision Drivers

- Career signal for Ottawa hiring. Authentication is the canonical technical-depth interview topic. A custom implementation built on well-understood primitives is a stronger signal than gluing together a magic library.
- Fine-grained control over the flow: magic-link login, refresh-token rotation with reuse detection, httpOnly cookies, multi-device sessions, eventual SSO for B2B. These requirements are specific enough that adapting an opinionated library tends to cost more than it saves.
- Privacy posture (PIPEDA, Loi 25, GDPR). No third-party service holding user identity data, no data residency outside our control, no vendor with access to email addresses or session metadata.
- No proprietary lock-in on an identity provider. A future migration of identity off Érable's servers should be a configuration change, not a rewrite.
- Acknowledged learning surface. Custom auth is listed in PROFILE_AND_LEARNING.md as a deliberate gap to close during this project.

## Considered Options

- A. Custom implementation on standard primitives (FastAPI backend, Next.js frontend, both built in-house)
- B. Better Auth
- C. Auth.js (NextAuth v5)
- D. Identity-as-a-service (Clerk, WorkOS, Auth0)
- E. Lucia v3+

## Decision Outcome

Chosen option: **A. Custom implementation on standard primitives**, because the project's privacy requirements, bilingual constraints, and career-signal goals all point in the same direction, and because the chosen primitives (signed JWTs, argon2id, secure cookies, refresh rotation) are well-documented and individually battle-tested.

The acceptance of this option is conditional on a documented threat model and an explicit set of security tests, both of which are owned artifacts (see Mitigations below).

### Consequences

- Good: complete control over the session lifecycle, token shape, claims, and rotation policy. Adding magic-link, multi-device sessions, or future SSO is additive work, not adaptation work.
- Good: no third-party dependency for identity. User emails and session metadata never leave our infrastructure.
- Good: strong career signal. The implementation is a defensible artifact in interviews — a custom refresh-rotation flow with reuse detection is exactly the kind of code that surfaces depth.
- Good: no library upgrade risk. The primitives (JWT libraries, argon2 implementations, cookie semantics) are stable; the assembly is ours and changes only when we change it.
- Bad: larger security surface than a vetted library. Bugs in our code are bugs in our auth. We mitigate this with the threat model, security test suite, and explicit security review before production (see Mitigations).
- Bad: more code to write and maintain than option B or E. Estimated 1.5 to 2 weeks of focused work spread across weeks 4-6, on top of the FastAPI lacuna already being closed.
- Bad: features we get for free with libraries (account linking, OAuth providers, MFA, device management) are roadmap items we will build incrementally rather than turn on.
- Bad: the authentication implementation must be reviewed by an external security-aware developer before production launch. This is a real cost, not optional.

### Chosen flow at a glance

- Password hashing: argon2id with parameters tuned to ~250ms on the production VPS (memory cost prioritized over time cost).
- Access tokens: signed JWT, 15-minute lifetime, HS256 with a strong secret rotated annually. Stored in React memory only, never in localStorage or sessionStorage.
- Refresh tokens: opaque random tokens (256 bits, base64url), 30-day lifetime, stored hashed in PostgreSQL alongside a `family_id` for rotation tracking. Issued in httpOnly Secure SameSite=Strict cookies.
- Refresh rotation: every refresh issues a new refresh token and invalidates the previous one. If a previously-rotated refresh token is presented (theft signal), the entire family is revoked and the user is logged out across all devices.
- Magic links: single-use tokens, 15-minute expiration, rate-limited to 5 per hour per email. Token is a signed payload (email + nonce + expiry), validated server-side, deleted on use.
- CSRF: SameSite=Strict on the refresh cookie plus a separate CSRF token for state-changing endpoints.
- Rate limiting: 5 login attempts per 15 minutes per IP and per account, applied at the FastAPI layer with Redis as the counter store.

### Mitigations for the larger security surface

- A threat model document (`docs/security/threat-model.md`) enumerates the assets, actors, attack surfaces, and mitigations. Written before the auth implementation lands in main.
- A dedicated security test suite covers timing-safe comparisons, refresh rotation invariants, magic-link single-use enforcement, rate limiting, and CSRF behavior.
- An external security-aware developer reviews the auth code before the production launch in week 10. Cost budgeted explicitly.
- Dependencies (argon2, JWT library, cookie parsers) are pinned and watched via Dependabot.
- Logs do not include emails, tokens, or any PII (per STACK_AND_CONVENTIONS.md privacy rules).

## Pros and Cons of the Options

### Option A: Custom implementation on standard primitives

- Good: maximum control. Magic-link, refresh rotation with reuse detection, multi-device sessions, and future SSO are all native, not adapted.
- Good: zero third-party identity service. Privacy promises in PRIVACY.md are verifiable from the codebase.
- Good: strongest career signal of the five options. The implementation itself is the artifact a senior interviewer can probe.
- Good: closes a deliberate learning gap.
- Bad: largest implementation effort and largest security surface in our code.
- Bad: requires explicit security review before production, which is an additional cost and dependency.
- Neutral: maintenance is ours forever, which is fine for a focused MVP and a foreseeable B2B SSO addition.

### Option B: Better Auth

- Good: the most modern Node-side auth library at the time of writing. Active development, sensible defaults, plugin architecture for OAuth, magic links, MFA.
- Good: handles edge cases (account linking, session invalidation, device management) that we would otherwise build ourselves.
- Bad: TypeScript-first and Node-centric. Our backend is Python (FastAPI), so Better Auth would only cover the frontend half. Splitting the auth implementation across two ecosystems (Better Auth on the Next.js side, custom on the FastAPI side) doubles the maintenance surface and creates a coordination boundary inside the most security-sensitive subsystem.
- Bad: opinionated session model that does not match our refresh-rotation-with-reuse-detection requirement out of the box. Adapting it costs time we would rather spend building the rotation flow ourselves and understanding it fully.
- Bad: career-signal-wise, "we used Better Auth" is a weaker answer in an interview than "we built refresh rotation with reuse detection on signed JWTs and argon2id, and here is the threat model".
- Neutral: a defensible choice for a TypeScript monolith. It is not our shape.

### Option C: Auth.js (NextAuth v5)

- Good: the historical default for Next.js authentication. Large community, many integrations.
- Good: OAuth providers come for free.
- Bad: same backend-language mismatch as Better Auth. Auth.js sits on the Next.js side; the FastAPI backend would still need its own session validation logic, leading to two sources of truth for what a "session" means.
- Bad: opinionated and not always cleanly composable with custom session models. Magic-link and refresh rotation with reuse detection both require fighting the framework.
- Bad: career signal is moderate. Many Next.js projects use Auth.js, so it does not differentiate.
- Neutral: a fine choice for a Next.js-only project with simple OAuth needs. Not our shape.

### Option D: Identity-as-a-service (Clerk, WorkOS, Auth0)

- Good: fastest time to first authenticated user. Production-grade flows, MFA, device management, and SSO out of the box.
- Good: security review of the core auth flow is the vendor's responsibility.
- Bad: sends user identity (emails, names, sometimes more) to a third party. Directly conflicts with the privacy posture in PRIVACY.md and the conformity drivers (PIPEDA, Loi 25, GDPR data residency arguments).
- Bad: ongoing per-MAU cost that scales with growth and that becomes meaningful at the projected scale.
- Bad: career signal is the weakest of the five. "We used Clerk" closes the conversation rather than opening it.
- Bad: vendor lock-in. Migrating off Clerk or Auth0 later is a real project, not a config change.
- Neutral: the right answer for a non-privacy-sensitive product with a small team and immediate revenue pressure. Not our shape.

### Option E: Lucia v3+

- Good: minimal, primitive-focused. Closer in spirit to "build it yourself" than the other libraries.
- Good: explicit session model rather than opaque magic.
- Bad: still TypeScript/Node-side, which leaves the FastAPI half uncovered.
- Bad: the original Lucia project was effectively wound down by its maintainer, with the recommendation to read the source and build your own. Subsequent forks and successors exist but the ecosystem is in flux.
- Bad: smaller community and less security review than the larger libraries.
- Neutral: philosophically closest to our chosen path, but the path itself dominates: if we are going to assemble auth from primitives, we may as well do it in the language we operate the backend in (Python) and keep one ecosystem.

## More Information

- Related ADRs: ADR-001 (FastAPI + ARQ) constrains the backend to Python; this is a precondition of the option-A reasoning. Future ADRs may cover B2B SSO (SAML/OIDC) once the white-label tier is live.
- References:
  - OWASP ASVS, authentication and session management chapters.
  - OWASP Cheat Sheets: Authentication, Session Management, JWT.
  - RFC 7519 (JWT), RFC 6265bis (cookies), argon2 RFC 9106.
  - "Refresh Token Rotation" pattern as documented by Auth0 engineering blog and IETF OAuth 2.1 draft.
- Revisit trigger: revisit this decision if any of the following occur:
  - A security review or external audit identifies systemic issues with the custom implementation.
  - B2B SSO requirements (SAML 2.0 federation, complex IdP integrations) reach a complexity that justifies a dedicated identity service or a federation gateway.
  - The team grows beyond solo and the maintenance cost of custom auth outweighs migration cost to a vetted library or service.
  - A library reaches maturity that materially closes the gap (covers Python backend + Next.js frontend with refresh rotation, reuse detection, and privacy posture compatible with PIPEDA/Loi 25).
