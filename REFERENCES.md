# References

Canonical documentation URLs for every technology in the Resonance Team stack.
Links point to the latest stable docs (no version pin in URL).
For prose and decision context, see [docs/tech-stack.md](./docs/tech-stack.md).

---

## Frontend — Web

| Technology | Docs | SSOT / ADR |
|---|---|---|
| Next.js (App Router) | https://nextjs.org/docs | [tech-stack.md](./docs/tech-stack.md#web-details) |
| TypeScript | https://www.typescriptlang.org/docs/ | [ADR 0001](./docs/adr/0001-typescript-for-web-and-zalo.md) |
| Tailwind CSS | https://tailwindcss.com/docs | [tech-stack.md](./docs/tech-stack.md#web-details) |
| TanStack Query (React Query) | https://tanstack.com/query/latest/docs | [tech-stack.md](./docs/tech-stack.md#web-details) |
| Zustand | https://zustand.docs.pmnd.rs/ | [tech-stack.md](./docs/tech-stack.md#web-details) |
| pnpm | https://pnpm.io/ | [tech-stack.md](./docs/tech-stack.md#web-details) |
| next-intl | https://next-intl-docs.vercel.app/ | [i18n.md](./docs/i18n.md#web-nextjs) |

## Frontend — Mobile

| Technology | Docs | SSOT / ADR |
|---|---|---|
| Flutter | https://docs.flutter.dev/ | [tech-stack.md](./docs/tech-stack.md#mobile-details-flutter) |
| Dart | https://dart.dev/guides | [tech-stack.md](./docs/tech-stack.md#mobile-details-flutter) |
| SwiftUI | https://developer.apple.com/documentation/swiftui | [tech-stack.md](./docs/tech-stack.md#mobile-details-swift-native) |
| Riverpod | https://riverpod.dev/docs | [tech-stack.md](./docs/tech-stack.md#mobile-details-flutter) |
| Dio | https://pub.dev/packages/dio | [tech-stack.md](./docs/tech-stack.md#mobile-details-flutter) |
| mocktail | https://pub.dev/packages/mocktail | [testing.md](./docs/testing.md#dart-flutter-mobile) |
| flutter_localizations | https://docs.flutter.dev/ui/accessibility-and-localization/internationalization | [i18n.md](./docs/i18n.md#mobile-flutter) |

## Frontend — Zalo Mini App

| Technology | Docs | SSOT / ADR |
|---|---|---|
| ZMP SDK | https://developers.zalo.me/docs/mini-app | [tech-stack.md](./docs/tech-stack.md#zalo-mini-app-details) |

## Backend — NestJS API

| Technology | Docs | SSOT / ADR |
|---|---|---|
| NestJS | https://docs.nestjs.com/ | [ADR 0003](./docs/adr/0003-nestjs-api-fastapi-ai-split.md) |
| Passport.js | https://www.passportjs.org/docs/ | [tech-stack.md](./docs/tech-stack.md#nestjs-api-details) |
| class-validator | https://github.com/typestack/class-validator | [tech-stack.md](./docs/tech-stack.md#nestjs-api-details) |
| class-transformer | https://github.com/typestack/class-transformer | [tech-stack.md](./docs/tech-stack.md#nestjs-api-details) |
| Prisma | https://www.prisma.io/docs | [tech-stack.md](./docs/tech-stack.md#nestjs-api-details) |
| Drizzle | https://orm.drizzle.team/docs/overview | [tech-stack.md](./docs/tech-stack.md#nestjs-api-details) |
| Pino (nestjs-pino) | https://getpino.io/#/ | [observability.md](./docs/observability.md#logs--loki--structured-json) |
| @willsoto/nestjs-prometheus | https://github.com/willsoto/nestjs-prometheus | [observability.md](./docs/observability.md#metrics--prometheus) |
| NestJS ThrottlerModule | https://docs.nestjs.com/security/rate-limiting | [security.md](./docs/security.md#authentication) |
| @sentry/node | https://docs.sentry.io/platforms/node/ | [observability.md](./docs/observability.md#error-tracking--glitchtip) |

## Backend — FastAPI AI Service

| Technology | Docs | SSOT / ADR |
|---|---|---|
| FastAPI | https://fastapi.tiangolo.com/ | [ADR 0003](./docs/adr/0003-nestjs-api-fastapi-ai-split.md) |
| Python 3.13 | https://docs.python.org/3/ | [tech-stack.md](./docs/tech-stack.md#fastapi-ai-service-details) |
| uvicorn | https://github.com/encode/uvicorn | [tech-stack.md](./docs/tech-stack.md#fastapi-ai-service-details) |
| qdrant-client | https://github.com/qdrant/qdrant-client | [tech-stack.md](./docs/tech-stack.md#fastapi-ai-service-details) |
| loguru | https://loguru.readthedocs.io/en/stable/ | [observability.md](./docs/observability.md#logs--loki--structured-json) |
| prometheus-fastapi-instrumentator | https://github.com/trallnag/prometheus-fastapi-instrumentator | [observability.md](./docs/observability.md#metrics--prometheus) |
| httpx | https://www.python-httpx.org/ | [testing.md](./docs/testing.md#python-fastapi-ai-service) |
| sentry-sdk (Python) | https://docs.sentry.io/platforms/python/ | [observability.md](./docs/observability.md#error-tracking--glitchtip) |

## AI Layer

| Technology | Docs | SSOT / ADR |
|---|---|---|
| OpenAI API | https://platform.openai.com/docs | [ADR 0007](./docs/adr/0007-openai-compatible-pluggable-ai.md) |
| Anthropic API | https://docs.anthropic.com/en/docs | [ADR 0007](./docs/adr/0007-openai-compatible-pluggable-ai.md) |
| OpenRouter | https://openrouter.ai/docs | [ADR 0007](./docs/adr/0007-openai-compatible-pluggable-ai.md) |
| vLLM | https://docs.vllm.ai/en/latest/ | [ADR 0007](./docs/adr/0007-openai-compatible-pluggable-ai.md) |
| Ollama | https://github.com/ollama/ollama | [ADR 0007](./docs/adr/0007-openai-compatible-pluggable-ai.md) |
| litellm | https://docs.litellm.ai/ | [ADR 0007](./docs/adr/0007-openai-compatible-pluggable-ai.md) |

## Data Layer

| Technology | Docs | SSOT / ADR |
|---|---|---|
| PostgreSQL 17 | https://www.postgresql.org/docs/ | [ADR 0005](./docs/adr/0005-postgres-plus-dedicated-vector-db.md) |
| Qdrant | https://qdrant.tech/documentation/ | [ADR 0005](./docs/adr/0005-postgres-plus-dedicated-vector-db.md) |
| Redis | https://redis.io/docs/latest/ | [tech-stack.md](./docs/tech-stack.md#data-layer) |

## Infrastructure

| Technology | Docs | SSOT / ADR |
|---|---|---|
| Docker | https://docs.docker.com/ | [ADR 0004](./docs/adr/0004-self-hosted-docker-compose.md) |
| Docker Compose | https://docs.docker.com/compose/ | [infrastructure.md](./docs/infrastructure.md#docker-compose-structure) |
| Traefik | https://doc.traefik.io/traefik/ | [infrastructure.md](./docs/infrastructure.md#reverse-proxy--tls) |
| Caddy | https://caddyserver.com/docs/ | [infrastructure.md](./docs/infrastructure.md#reverse-proxy--tls) |
| Let's Encrypt | https://letsencrypt.org/docs/ | [infrastructure.md](./docs/infrastructure.md#reverse-proxy--tls) |
| Hetzner | https://docs.hetzner.com/ | [infrastructure.md](./docs/infrastructure.md#vps-provider) |
| Borg | https://www.borgbackup.org/ | [infrastructure.md](./docs/infrastructure.md#postgresql) |
| rclone | https://rclone.org/docs/ | [infrastructure.md](./docs/infrastructure.md#postgresql) |

## CI/CD

| Technology | Docs | SSOT / ADR |
|---|---|---|
| GitHub Actions | https://docs.github.com/en/actions | [ci-cd.md](./docs/ci-cd.md) |
| GitHub Container Registry (ghcr.io) | https://docs.github.com/en/packages/working-with-a-github-container-registry | [ci-cd.md](./docs/ci-cd.md) |
| Renovate | https://docs.renovatebot.com/ | [ci-cd.md](./docs/ci-cd.md#version-policy) |
| actions/checkout | https://github.com/actions/checkout | [ci-cd.md](./docs/ci-cd.md#github-actions-workflow-structure) |
| pnpm/action-setup | https://github.com/pnpm/action-setup | [ci-cd.md](./docs/ci-cd.md#github-actions-workflow-structure) |
| astral-sh/setup-uv | https://github.com/astral-sh/setup-uv | [ci-cd.md](./docs/ci-cd.md#github-actions-workflow-structure) |
| appleboy/ssh-action | https://github.com/appleboy/ssh-action | [ci-cd.md](./docs/ci-cd.md#github-actions-workflow-structure) |

## Observability

| Technology | Docs | SSOT / ADR |
|---|---|---|
| Prometheus | https://prometheus.io/docs/ | [observability.md](./docs/observability.md#metrics--prometheus) |
| Loki | https://grafana.com/docs/loki/latest/ | [observability.md](./docs/observability.md#logs--loki--structured-json) |
| Grafana | https://grafana.com/docs/grafana/latest/ | [observability.md](./docs/observability.md#grafana-dashboards) |
| GlitchTip | https://glitchtip.com/documentation | [observability.md](./docs/observability.md#error-tracking--glitchtip) |
| Promtail | https://grafana.com/docs/loki/latest/send-data/promtail/ | [observability.md](./docs/observability.md#loki--promtail) |
| prom-client (Node.js) | https://github.com/siimon/prom-client | [observability.md](./docs/observability.md#metrics--prometheus) |

## Testing

| Technology | Docs | SSOT / ADR |
|---|---|---|
| vitest | https://vitest.dev/guide/ | [testing.md](./docs/testing.md#typescript-web--nestjs-api--zalo) |
| Playwright | https://playwright.dev/docs/intro | [testing.md](./docs/testing.md#playwright-for-e2e) |
| msw | https://mswjs.io/docs/ | [testing.md](./docs/testing.md#typescript-web--nestjs-api--zalo) |
| pytest | https://docs.pytest.org/en/stable/ | [testing.md](./docs/testing.md#python-fastapi-ai-service) |
| pytest-asyncio | https://pytest-asyncio.readthedocs.io/en/stable/ | [testing.md](./docs/testing.md#python-fastapi-ai-service) |

## Linting & Formatting

| Technology | Docs | SSOT / ADR |
|---|---|---|
| ESLint | https://eslint.org/docs/latest/ | [workflow.md](./docs/workflow.md#tools) |
| Prettier | https://prettier.io/docs/ | [workflow.md](./docs/workflow.md#tools) |
| ruff | https://docs.astral.sh/ruff/ | [testing.md](./docs/testing.md#python-fastapi-ai-service) |
| pyright | https://github.com/microsoft/pyright | [testing.md](./docs/testing.md#python-fastapi-ai-service) |

## Git Hooks & Commit Lint

| Technology | Docs | SSOT / ADR |
|---|---|---|
| husky | https://typicode.github.io/husky/ | [workflow.md](./docs/workflow.md#tools) |
| lint-staged | https://github.com/okonet/lint-staged | [workflow.md](./docs/workflow.md#tools) |
| commitlint | https://commitlint.js.org/ | [workflow.md](./docs/workflow.md#tools) |

## Monorepo

| Technology | Docs | SSOT / ADR |
|---|---|---|
| Turborepo | https://turbo.build/repo/docs | [ADR 0008](./docs/adr/0008-turborepo-with-python-shim.md) |
| pnpm workspaces | https://pnpm.io/workspaces | [repositories.md](./docs/repositories.md#default-turborepo-monorepo) |

## Dev Tooling

| Technology | Docs | SSOT / ADR |
|---|---|---|
| uv | https://docs.astral.sh/uv/ | [repositories.md](./docs/repositories.md#python-in-the-monorepo--the-packagejson-shim) |
| git-crypt | https://github.com/AGWA/git-crypt | [security.md](./docs/security.md#git-crypt-rules) |

---

> To add a new technology: add an entry to the appropriate table and, if it changes a
> stack default, open an issue and add an ADR per [CONTRIBUTING.md](./CONTRIBUTING.md).
