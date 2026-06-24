# ADR 0007: OpenAI-compatible, provider-pluggable AI layer

- **Date:** 2026-06-24
- **Status:** accepted

## Context

The team builds AI applications that depend on LLM inference. Provider
lock-in is a risk: pricing changes, model deprecations, or regional access
issues (relevant for Vietnam) could force a swap.

## Decision

The FastAPI AI service uses an **OpenAI-compatible client interface** with
a **provider routing module**. The active provider is selected via
environment variable (`AI_PROVIDER`, `AI_BASE_URL`, `AI_API_KEY`), not
hardcoded into any service.

Default providers supported out of the box:
- OpenAI (the API shape everything else follows)
- Anthropic (via litellm proxy or a compatible adapter)
- OpenRouter (unified billing across many models)
- Self-hosted vLLM / Ollama (local models, no cloud dependency)

The NestJS API never calls an LLM directly. All AI traffic goes through
FastAPI for observability, retries, and provider-agnostic routing.

## Consequences

- **Positive:** Swap providers without touching NestJS or client code.
  Change three environment variables; restart FastAPI.
- **Positive:** Vietnam-friendly: switch to a local provider or self-hosted
  model if OpenAI latency or access becomes an issue.
- **Positive:** A/B test providers in production (route X% of traffic to
  provider A, the rest to B) — built into the routing module.
- **Negative:** OpenAI-compatible is a lowest-common-denominator API shape.
  Provider-specific features (Anthropic's extended thinking, tool use
  variants) may not be available through the compatible interface without
  adapter work.
- **Negative:** The litellm proxy layer adds a dependency and an extra
  latency hop if used for Anthropic routing.
