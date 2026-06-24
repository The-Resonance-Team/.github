# Tech Stack

The Resonance Team's standard technology stack. New applications default to
these choices unless an ADR overrides them.

> **Principle:** pick reasonable defaults, codify them, and move on. Debate
> belongs in an ADR, not at the start of every project.

## Frontends

| Surface | Framework | Language | Deploy | Default? |
|---|---|---|---|---|
| Web | Next.js (App Router) | TypeScript | Docker on VPS | Yes |
| Mobile (primary) | Flutter | Dart | App Store + Play Store | Yes |
| Mobile (native) | SwiftUI | Swift | App Store | Exception only |
| Zalo Mini App | ZMP SDK | JavaScript / TS | Zalo platform | VN-market only |

### Web details

- **Framework:** Next.js, App Router, RSC, TypeScript strict mode.
- **Styling:** Tailwind CSS v4.
- **State:** React Query (TanStack) for server state; Zustand for
  client-only state.
- **Build:** `next build` → Docker image (standalone output mode).
- **Package manager:** pnpm.

### Mobile details (Flutter)

- **Framework:** Flutter, Dart, Material 3.
- **State:** Riverpod.
- **Networking:** Dio + shared API client types from `@resonance/shared`.
- **Deploy:** CI builds IPA/AAB → App Store Connect / Play Console.

### Mobile details (Swift native)

- Used only when Flutter cannot reach a platform capability (AR, deep camera,
  wallet, widgets, App Clips). Document the reason in the project ADR.

### Zalo Mini App details

- **Platform:** ZMP SDK (HTML/JS), TypeScript optional.
- Used for VN-market products where Zalo reach matters. Not a default for
  every product.

## Backend services

| Service | Framework | Language | Runtime | Role |
|---|---|---|---|---|
| API + Auth | NestJS | TypeScript | Node.js 24 LTS | Client-facing REST/GraphQL, auth (Passport.js), business logic, validation |
| AI service | FastAPI | Python 3.13 | Docker | LLM inference, embeddings, RAG, model orchestration |

### NestJS API details

- **Modules:** structured by domain, not by technical layer.
- **Auth:** Passport.js strategies (JWT + OAuth2). Sessions stateless by
  default.
- **Validation:** class-validator + class-transformer (DTOs).
- **Database access:** typed ORM (Prisma or Drizzle — per-project choice;
  Drizzle preferred for monorepo TS alignment).
- **Testing:** vitest (share config with web).

### FastAPI AI service details

- **Provider pattern:** OpenAI-compatible client interface. Provider
  (OpenAI, Anthropic via adapter, OpenRouter, local vLLM/Ollama) chosen per
  deployment via `config.yaml` or env var.
- **Streaming:** SSE for streaming responses to the NestJS layer.
- **Vector store:** Qdrant client (`qdrant-client`).
- **Testing:** pytest + pytest-asyncio.

## AI layer

OpenAI-compatible by default. No lock-in to one provider.

```
FastAPI AI service
  └─ config.py: resolve provider from env $AI_PROVIDER/$AI_BASE_URL/$AI_API_KEY
     ├─ OpenAI (default target)
     ├─ Anthropic (via compatible adapter or litellm proxy)
     ├─ OpenRouter (unified billing)
     ├─ vLLM / Ollama (self-hosted, local GPU)
     └─ litellm proxy (multi-model routing)
```

The NestJS API **does not** call LLMs directly. All AI traffic goes through
the FastAPI service for observability, retries, and provider swapping.

## Data layer

| Component | Technology | Notes |
|---|---|---|
| Primary DB | PostgreSQL 17 | Self-hosted on VPS or managed |
| Vector DB | Qdrant (default) | Dedicated; Milvus/Weaviate as scale alts |
| Cache / Queue | Redis (optional) | Added when a product needs it |

## Infrastructure

| Layer | Technology |
|---|---|
| Host | VPS provider (self-managed) |
| Container runtime | Docker + Docker Compose |
| Reverse proxy | Traefik or Caddy (automatic TLS via Let's Encrypt) |
| TLS | Let's Encrypt (automatic via reverse proxy) |
| Region | Singapore (low latency to VN) |

Full details: [infrastructure.md](infrastructure.md).

## CI/CD

| Stage | Tool |
|---|---|
| CI | GitHub Actions (lint, test, build per PR) |
| CD | SSH + Docker Compose (`pull && up -d`) |
| Registry | GitHub Container Registry (ghcr.io) |
| Secrets injection | `.env` synced from password manager → CI → VPS |

Full details: [ci-cd.md](ci-cd.md).

## Observability

| Signal | Tool |
|---|---|
| Metrics | Prometheus |
| Logs | Loki (aggregated), structured JSON (Pino/loguru) |
| Dashboards + Alerts | Grafana |
| Error tracking | GlitchTip (self-hosted, Sentry-compatible SDKs) |

Full details: [observability.md](observability.md).

## Secrets management

`.env` files (gitignored) + `git-crypt` for committed secrets. Discipline
rules are mandatory (see [security.md](security.md)).

## Testing (per language)

| Language | Unit / Integration | E2E |
|---|---|---|
| TypeScript | vitest | Playwright |
| Python | pytest + pytest-asyncio | — (integration via vitest calling the service) |
| Dart | flutter test + integration_test | — |

Full details: [testing.md](testing.md).

## Internationalization

Vietnamese + English as default language pair.

| Surface | Library |
|---|---|
| Web (Next.js) | next-intl |
| Mobile (Flutter) | flutter_localizations + intl |
| Zalo | Built-in i18n via ZMP API |

Full details: [i18n.md](i18n.md).

## Repository topology

| Pattern | Detail |
|---|---|
| Monorepo tool | Turborepo |
| TS family | `apps/web`, `apps/api` (NestJS), `apps/zalo` (optional), `packages/shared` |
| Python AI | Live in monorepo as workspace dir with a thin `package.json` shim running `uv`/`pytest`; turbo caches via `dependsOn` |
| Flutter | Separate repo (Dart tooling doesn't benefit from Turborepo) |
| Shared types | `packages/shared` → published as `@resonance/shared`; Flutter consumes via a generated Dart client |

Full details: [repositories.md](repositories.md).

## Engineering workflow

| Practice | Default |
|---|---|
| Branching | GitHub Flow (short-lived feature branches off `main`) |
| Commits | Conventional Commits (`feat:`, `fix:`, `chore:`, etc.) |
| PR merge | Squash-merge, one review required |
| Issues | Issue templates (bug, feature, config), triage labels from dotfiles |
| Docs | ADRs for all architectural decisions |

Full details: [workflow.md](workflow.md).

## Version policy

| Layer | Stability |
|---|---|
| Node.js | 24 LTS |
| Python | 3.13 |
| PostgreSQL | 17 |
| All packages | pin major versions; Renovate for automated bumps |

---

> **To change a default:** open an issue, propose the change, add an ADR
> superseding the prior one. See [CONTRIBUTING.md](../CONTRIBUTING.md).
