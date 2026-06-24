# ADR 0003: NestJS API + FastAPI AI split

- **Date:** 2026-06-24
- **Status:** accepted

## Context

The team builds AI applications with a significant LLM inference and RAG
component. The backend must serve both traditional REST endpoints (auth,
CRUD, business logic) and AI endpoints (chat, embeddings, retrieval).

## Decision

Split the backend into two services:

- **NestJS (TypeScript)** — client-facing API. Owns authentication
  (Passport.js), authorization, business logic, CRUD, and API contracts.
  Clients (web, mobile, Zalo) talk to NestJS exclusively.

- **FastAPI (Python 3.13)** — internal AI service. Owns LLM inference,
  embedding generation, vector search, RAG pipelines, and provider routing.
  Only NestJS calls FastAPI (over the internal Docker network).

## Consequences

- **Positive:** Each service uses the ecosystem best suited to its job
  (TS for API contracts + auth, Python for AI/ML libraries).
- **Positive:** AI service can scale independently — deploy on GPU instances
  while the API runs on cheap compute.
- **Positive:** Provider swaps (OpenAI → self-hosted vLLM) touch only
  the FastAPI service; NestJS is unaffected.
- **Negative:** Two services with different languages, runtimes, and test
  frameworks add mental overhead. The monorepo with the `package.json`
  shim pattern keeps them discoverable together.
- **Negative:** The NestJS → FastAPI hop adds latency. Mitigated by
  keeping them on the same VPS/Docker network.
