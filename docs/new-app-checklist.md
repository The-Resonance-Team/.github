# New App Checklist

Step-by-step bootstrap recipe for starting a new Resonance Team application.
Run through this checklist when kicking off a new product.

## Phase 1 — Scaffold

- [ ] Create a new repo under `The-Resonance-Team` on GitHub.
- [ ] Clone and init Turborepo:
  ```bash
  npx create-turbo@latest my-product -e minimal
  ```
- [ ] Set up `pnpm-workspace.yaml`:
  ```yaml
  packages:
    - "apps/*"
    - "packages/*"
    - "services/*"
  ```
- [ ] Add workspace dirs: `apps/web`, `apps/api`, `services/ai`,
  `packages/shared`, `tools/eslint-config`.

## Phase 2 — Shared types

- [ ] Create `packages/shared/` — Zod schemas + TS types for API contracts.
- [ ] Export `@resonance/shared` from `packages/shared/package.json`.
- [ ] Add `"@resonance/shared": "workspace:*"` to `apps/web` and `apps/api`
  dependencies.

## Phase 3 — Web (Next.js)

- [ ] Init Next.js app in `apps/web/`:
  ```bash
  cd apps/web && npx create-next-app@latest . --ts --app --tailwind --eslint --src-dir
  ```
- [ ] Add `next-intl` for i18n (VN + EN).
- [ ] Add TanStack Query for server state.
- [ ] Add vitest for testing.

## Phase 4 — API (NestJS)

- [ ] Init NestJS in `apps/api/`:
  ```bash
  npx @nestjs/cli new apps/api --package-manager pnpm --strict
  ```
- [ ] Wire Passport.js auth module (JWT strategy).
- [ ] Add `@willsoto/nestjs-prometheus` for metrics.
- [ ] Add `@sentry/node` for error tracking (point to GlitchTip DSN).
- [ ] Add vitest (share config with web via `tools/vitest-config`).

## Phase 5 — AI service (FastAPI)

- [ ] Create `services/ai/` with `pyproject.toml` using uv:
  ```bash
  cd services/ai && uv init
  uv add fastapi uvicorn openai qdrant-client loguru prometheus-fastapi-instrumentator sentry-sdk
  uv add --dev pytest pytest-asyncio ruff pyright
  ```
- [ ] Add the `package.json` shim so turbo can run AI tasks.
- [ ] Add `services/ai` to `turbo.json` tasks.

## Phase 6 — Data

- [ ] Add PostgreSQL to `docker-compose.yml` (image: `postgres:17-alpine`).
- [ ] Add Qdrant to `docker-compose.yml` (image: `qdrant/qdrant:latest`).
- [ ] Add Redis if the product needs caching or queues (optional).
- [ ] Set up Prisma or Drizzle in `apps/api`.
- [ ] Create initial migration.
- [ ] Add a backup script (`pg_dump` + rclone to S3 or Borg).

## Phase 7 — Infrastructure

- [ ] Provision a VPS (Hetzner, Singapore region, or another provider).
- [ ] Install Docker + Docker Compose on the VPS.
- [ ] Set up Traefik or Caddy as reverse proxy with Let's Encrypt TLS.
- [ ] Create `docker-compose.yml` + `docker-compose.prod.yml`.
- [ ] Create `.env.example` documenting all required env vars.
- [ ] Create the real `.env` on the VPS (values from password manager).
- [ ] Configure firewall (ufw: allow 80/443 + SSH port).
- [ ] Set up the observability stack (Prometheus, Loki, Grafana, GlitchTip).
- [ ] Verify: `docker compose up -d` → all services healthy on `/health`.

## Phase 8 — CI/CD

- [ ] Copy the org-wide CI template or create `.github/workflows/ci.yml`.
- [ ] Create `.github/workflows/deploy.yml` with VPS SSH secrets.
- [ ] Add GitHub repository secrets: `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`.
- [ ] Configure Renovate for automated dependency updates.
- [ ] Push → verify CI runs on PR → merge → verify deploy.

## Phase 9 — Verify production readiness

- [ ] All services expose `/health` with dependency checks.
- [ ] All services expose `/metrics` (Prometheus scraping confirmed).
- [ ] Structured JSON logs visible in Grafana (Loki datasource).
- [ ] GlitchTip receives errors (test by hitting a non-existent route).
- [ ] Grafana dashboard shows API request rate and error rate.
- [ ] `.env` NOT committed (verify with `git ls-files | grep .env`).
- [ ] `.env.example` committed and complete.
- [ ] ADR log started (`docs/adr/0001-*.md` with project-specific decisions).

## Phase 10 — Ship

- [ ] Set up DNS: `api.myproduct.com` and `myproduct.com` pointing to VPS IP.
- [ ] Verify TLS (green lock on all domains).
- [ ] `README.md` updated with project overview and links to this checklist.

---

> **Tip:** Not every product needs every phase immediately. Mobile and Zalo
> surfaces can be added later. The minimum deployable product = Phase 1–8
> (web + API + AI + data + infra + CI). Mobile is Phase N+1.
