# Architecture

System shape, boundaries, and data flow for a Resonance Team application.

## High-level shape

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│   Web    │  │  Mobile  │  │  Zalo    │   ← presentational clients
│ Next.js  │  │ Flutter  │  │ ZMP SDK  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │              │              │
     └──────────────┼──────────────┘
                    │  HTTPS / REST / GraphQL
              ┌─────┴─────┐
              │  NestJS    │  ← app API + auth + business logic
              │  API       │
              └─────┬─────┘
                    │  HTTP (internal) / gRPC
              ┌─────┴─────┐
              │  FastAPI   │  ← AI inference, embeddings, RAG
              │  AI svc    │
              └─────┬─────┘
                    │
         ┌──────────┼──────────┐
         │          │          │
    ┌────┴───┐ ┌───┴────┐ ┌───┴───┐
    │Postgres│ │ Qdrant │ │ Redis │  ← data (Redis optional)
    └────────┘ └────────┘ └───────┘
```

## Service boundaries

### NestJS API — the app gateway

- **Owns:** auth (sessions / JWT), user management, CRUD operations, business
  logic, request validation, rate limiting, API contracts.
- **Does NOT own:** LLM inference, embeddings, RAG orchestration.
- **Talks to:** Postgres (direct), FastAPI (HTTP), Redis (if present).

### FastAPI AI service — the intelligence

- **Owns:** LLM calls, prompt templates, embedding generation, vector search,
  RAG pipeline, streaming responses, provider routing.
- **Does NOT own:** auth, user data, business rules.
- **Talks to:** Postgres (optional, for job state), Qdrant (direct), configured
  AI provider (OpenAI-compatible endpoint).

### Why the split?

- Python is the default ecosystem for AI/ML libraries. NestJS is the
  better framework for REST APIs, auth, and business logic.
- The AI service can scale independently (GPU instances vs regular compute).
- Provider changes (swap OpenAI for a self-hosted model) touch only the
  FastAPI service — the NestJS front never changes.
- See [ADR 0003](adr/0003-nestjs-api-fastapi-ai-split.md).

## Request lifecycle

```
Client request
  → Traefik/Caddy (TLS termination)
    → NestJS API (auth check, validation)
      → (if AI needed) FastAPI AI service
        → {OpenAI-compatible provider} or {Qdrant}
      ← streaming SSE or JSON response
    ← JSON / streaming response
  ← client
```

## Auth flow

```
Client
  → POST /auth/login (credentials or OAuth2 redirect)
    → NestJS + Passport strategy
      ← JWT (access + refresh) or session cookie
  → subsequent requests: Authorization: Bearer <jwt>
    → NestJS guard validates → injects user context
```

No auth in the FastAPI layer — it trusts the NestJS caller (internal network,
mutual TLS or shared secret header).

## Data flow for AI

```
Client: "summarize this document"
  → NestJS: POST /ai/summarize { document_id }
    → NestJS: load document from Postgres
    → NestJS: POST FastAPI /summarize { text, ... }
      → FastAPI: chunk text → embed chunks → query Qdrant for context
      → FastAPI: assemble prompt → call LLM provider
      ← FastAPI: SSE stream { token }
    ← NestJS: relay SSE stream to client
  ← Client: rendered tokens
```

## Deployment topology

```
VPS (Singapore)
├── traefik/caddy (80/443, auto-TLS)
├── nestjs-api       (port 3000, internal)
├── fastapi-ai       (port 8000, internal)
├── postgres         (port 5432, internal)
├── qdrant           (port 6333, internal)
├── redis            (port 6379, internal, optional)
├── prometheus       (port 9090, internal)
├── loki             (port 3100, internal)
├── grafana          (port 3001, internal)
└── glitchtip        (port 8001, internal)
```

All inter-service communication stays within the Docker Compose network.
Only Traefik/Caddy is exposed to the internet.

## Cross-cutting concerns

| Concern | Implementation |
|---|---|
| Logging | Structured JSON (Pino for NestJS, loguru for FastAPI) → Loki |
| Metrics | Prometheus exporters (each service exposes `/metrics`) → Grafana dashboards |
| Tracing | OpenTelemetry (future; start with correlation IDs in headers) |
| Error tracking | GlitchTip SDKs (Sentry-compatible) in all services |
| Secrets | Injected via `.env` mounted at deploy; never in images |
