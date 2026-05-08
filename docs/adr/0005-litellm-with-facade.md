# ADR-005: LiteLLM with a thin in-house facade for LLM access

## Status

Accepted — 2026-05-08

## Context and Problem Statement

Érable's product surface depends on LLM calls in several places: the chat assistant for newcomer questions, RAG over indexed Canadian source material, structured outputs for the CV builder and cover letter generator, and the eventual ECA concierge agent. From day one of development, two providers must coexist: Anthropic Claude in production (Sonnet for chat, Haiku for simple tasks) and Ollama running locally (Mistral 7B Q4 + BGE-M3 embeddings) during development to keep iteration costs near zero.

Two architectural questions need answers before any LLM-touching code lands:

1. Should LLM calls go through an abstraction layer, or should code call provider SDKs directly?
2. If an abstraction layer is needed, what does it look like?

These cannot be deferred. Every piece of code that talks to a model — chat endpoints, embedding pipelines, structured-output parsers — depends on the answer.

## Decision Drivers

- Local development on Ollama with no Anthropic API spend during prompt iteration. Confirmed by stack conventions; this is non-negotiable.
- Symmetry between dev and prod: code should not know whether the model behind the call is Claude Sonnet on Anthropic or Mistral 7B on a local Ollama daemon, beyond a configuration flag.
- Future-proofing for production resilience. Anthropic outages happen; an OpenAI fallback in production is a phase-2 capability we want to enable later without rewriting calls.
- Ecosystem alignment with the rest of the LLM tooling we plan to adopt: Langfuse for observability (S9), prompt caching, cost tracking. The LLM gateway layer should integrate with these tools naturally.
- Career signal, secondary driver. LiteLLM is the dominant LLM gateway in the Python ecosystem in 2026 and is recognized by interviewers in AI-adjacent roles. Familiarity with it is a credible CV signal, though weaker than mastery of the patterns above the gateway (RAG, agents, structured outputs).
- Avoid framework lock-in. Any abstraction we adopt must remain swappable. Code in `services/` and `rag/` calls our facade, not LiteLLM directly.

## Considered Options

- A. LiteLLM library mode + thin in-house facade (`integrations/llm/`)
- B. Anthropic SDK direct, no abstraction layer
- C. LangChain LLM abstractions
- D. Custom abstraction without LiteLLM

## Decision Outcome

Chosen option: **A. LiteLLM library mode + thin in-house facade**, because it is the only option that solves the Ollama-dev / Anthropic-prod symmetry today, leaves the door open to multi-provider production resilience tomorrow, and keeps the rest of the codebase decoupled from any specific gateway library through the in-house facade.

The decision is made at three coupled levels, all answered together:

1. **An abstraction layer is required.** Direct provider SDKs would force the codebase to know whether it's talking to Anthropic, Ollama, or any future provider, which breaks the dev/prod symmetry that justifies Ollama in the first place.
2. **LiteLLM is that layer.** It is the de facto standard, supports both Anthropic and Ollama natively, and aligns with the wider ecosystem (Langfuse, cost tracking, structured outputs).
3. **A thin facade wraps LiteLLM.** All Érable code calls our facade, not `litellm.completion()` directly. The facade is small (estimated 50-150 lines), exposes a Pydantic-typed API tailored to our needs, and centralizes observability hooks, retry policy, and provider routing config. This protects us from future LiteLLM API changes and from a possible migration off LiteLLM.

### Consequences

- Good: identical code path for dev (Ollama) and prod (Anthropic). Switching is one environment variable.
- Good: adding a provider later (OpenAI fallback, Cohere, Groq) is a configuration change plus a model-name string. No new SDK integration in business code.
- Good: native integration path with Langfuse for tracing, cost tracking, and prompt versioning when that work lands in S9.
- Good: in-house facade insulates Érable from LiteLLM's release cadence. If LiteLLM ships a breaking change or its trajectory shifts, only the facade needs updating, not every call site.
- Good: career signal. LiteLLM appears in many AI infra job descriptions in 2026; using it deliberately and explaining the wrapper rationale is defensible in interviews.
- Bad: one extra dependency to maintain. LiteLLM ships frequently and has occasionally broken compatibility with provider SDKs upstream. We pin versions and treat upgrades as deliberate.
- Bad: small abstraction tax. The facade is one more place to look when debugging an LLM call. The cost is real but bounded by keeping the facade thin.
- Bad: LiteLLM normalizes responses to OpenAI's shape. Anthropic-specific features (computer use, certain caching modes) may need facade-level escape hatches when we want to use them.

### Facade shape at a glance

The facade lives in `backend/src/erable/integrations/llm/`. Its surface is intentionally narrow:

```python
# integrations/llm/client.py

class LLMMessage(BaseModel):
    role: Literal["system", "user", "assistant"]
    content: str

class LLMResponse(BaseModel):
    text: str
    model: str
    input_tokens: int
    output_tokens: int

async def complete(
    messages: list[LLMMessage],
    *,
    model: str | None = None,         # defaults from settings
    max_tokens: int = 1024,
    temperature: float = 0.7,
    response_model: type[BaseModel] | None = None,  # for structured outputs
) -> LLMResponse | BaseModel:
    ...

async def embed(
    texts: list[str],
    *,
    model: str | None = None,         # defaults from settings
) -> list[list[float]]:
    ...
```

That is the entire public surface. Internally it dispatches to `litellm.acompletion()` and `litellm.aembedding()`, applies retry policy, attaches Langfuse tracing when enabled, and normalizes errors into `ErableException` subclasses.

The provider-and-model is selected by configuration, not by the caller. In development, settings resolve `ollama/mistral` and `ollama/bge-m3`; in production, `claude-sonnet-4-5` and a hosted BGE-M3 endpoint.

## Pros and Cons of the Options

### Option A: LiteLLM library mode + thin in-house facade

- Good: solves the Ollama-dev / Anthropic-prod symmetry without us writing the multi-provider adapter.
- Good: 100+ providers supported out of the box; future migrations or A/B tests cost a config line.
- Good: integrates with Langfuse, prompt caching helpers, and cost tracking. Aligns with our planned observability stack.
- Good: the in-house facade keeps LiteLLM swappable. If we later regret the choice, only the facade changes.
- Bad: dependency on a fast-moving library. Pinning and deliberate upgrades are required.
- Bad: thin abstraction over OpenAI's response shape. Provider-specific features may need facade-level escape hatches.
- Neutral: the facade is small but real ongoing maintenance. Cheap, not free.

### Option B: Anthropic SDK direct, no abstraction layer

- Good: simplest possible implementation. One SDK, one mental model, the cleanest types.
- Good: Anthropic's SDK is high quality, well-documented, and exposes provider-specific features cleanly.
- Bad: incompatible with the Ollama-in-dev requirement. Calling Ollama from code that imports `from anthropic import Anthropic` is not possible without a parallel code path, which is exactly the abstraction we are trying to avoid.
- Bad: makes future multi-provider work expensive. An OpenAI fallback in production becomes a refactor, not a config change.
- Neutral: a defensible choice for a project committed to a single provider end-to-end. Not our shape, given Ollama-in-dev is a hard requirement.

### Option C: LangChain LLM abstractions

- Good: large community, many integrations, established patterns.
- Good: built-in chains, agents, retrievers — useful as references even if we don't adopt them.
- Bad: opinionated framework that pulls in chains, agents, retrievers, memory abstractions, and a worldview about how LLM apps should be structured. Adopting LangChain even just for its LLM wrapper invites scope creep into the rest of the codebase.
- Bad: heavyweight dependency tree, frequent breaking changes between major versions, mixed reputation for production stability.
- Bad: career signal is mixed. LangChain familiarity is widespread but increasingly viewed with skepticism by senior engineers who have hit its abstractions in production.
- Neutral: explicitly excluded by stack conventions, which already documented LangChain as out of scope. This ADR confirms that exclusion at the gateway level.

### Option D: Custom abstraction without LiteLLM

- Good: zero external dependency for the gateway. Total control over every line.
- Good: stronger learning exercise. Writing the Anthropic and Ollama adapters by hand teaches the actual API shapes of each provider.
- Bad: scope creep is real. What looks like 50 lines becomes a few hundred once retries, structured outputs, streaming, error normalization, cost tracking, and a third provider land. Every line is ours to maintain.
- Bad: weaker integration with the wider ecosystem (Langfuse, prompt caching) — those tools have first-class LiteLLM support and would need glue code from us.
- Bad: career-signal-wise, "we wrote our own LLM gateway from scratch" is defensible but invites the next question: "why didn't you use LiteLLM?" A clear answer is needed; in our case, the clear answer is "we did, and here is the facade" — which points back to option A.
- Neutral: a credible path for a team with strong existing LLM-infra muscle memory or specific requirements LiteLLM cannot meet. Neither applies here.

## More Information

- Related ADRs:
  - ADR-001 (FastAPI + ARQ) — async LiteLLM calls (`acompletion`, `aembedding`) align with the async stack.
  - Future ADR on Langfuse integration (planned for S9) — will plug into this facade rather than bypass it.
  - Future ADR on RAG architecture (planned for S4) — embedding calls go through `embed()` in this facade.
- Excluded from this ADR (mentioned in passing only): Vercel AI SDK (TypeScript-first, our backend is Python), Portkey (commercial gateway, comparable functionality but the ecosystem alignment with LiteLLM is stronger in the open-source Python world).
- References:
  - LiteLLM documentation, particularly the `completion` and `embedding` API reference and the Ollama provider integration.
  - Langfuse integration notes for LiteLLM.
  - OpenAI chat completion API specification (LiteLLM normalizes to this shape).
- Revisit trigger: revisit this decision if any of the following occur:
  - LiteLLM's release stability degrades materially, or its maintenance trajectory becomes uncertain.
  - We need LiteLLM Proxy mode (separate gateway daemon shared across services), at which point the architecture changes from library-embedded to service-oriented.
  - Anthropic releases native multi-provider support in its SDK that covers our needs (unlikely but worth watching).
  - The facade grows past ~300 lines, suggesting we are working around LiteLLM more than with it.
