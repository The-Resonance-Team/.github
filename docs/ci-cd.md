# CI/CD

Continuous integration and deployment pipeline for a Resonance Team
application.

## Pipeline overview

```
PR opened → GitHub Actions
  ├── Lint (parallel per language)
  │   ├── web/api/zalo: ESLint + Prettier
  │   └── ai: ruff
  ├── Type-check (parallel)
  │   ├── TS: tsc --noEmit
  │   └── Python: pyright
  ├── Test (parallel)
  │   ├── TS: vitest
  │   ├── Python: pytest
  │   └── E2E: Playwright
  └── Build (only if test+lint pass)
      ├── nestjs-api → Docker image → ghcr.io
      ├── fastapi-ai → Docker image → ghcr.io
      └── web → Docker image → ghcr.io

Merge to main → Build + Deploy
  ├── (same build as PR, but tagged :latest and :<sha>)
  └── Deploy: SSH → VPS → docker compose pull && up -d
```

## GitHub Actions workflow structure

### CI (per PR) — `.github/workflows/ci.yml`

```yaml
name: CI
on:
  pull_request:
    branches: [main]

jobs:
  lint-ts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - run: pnpm lint

  lint-python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: cd services/ai && uv run ruff check .

  typecheck:
    runs-on: ubuntu-latest
    needs: [lint-ts]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - run: pnpm typecheck

  test-ts:
    runs-on: ubuntu-latest
    needs: [lint-ts]
    services:
      postgres:
        image: postgres:17-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports: [5432:5432]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - run: pnpm test -- --filter="./apps/*"

  test-python:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: cd services/ai && uv run pytest

  build:
    needs: [typecheck, test-ts, test-python]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - run: |
          docker build -t ghcr.io/${{ github.repository }}/api:${{ github.sha }} -f apps/api/Dockerfile .
          docker push ghcr.io/${{ github.repository }}/api:${{ github.sha }}
      # repeat for web, ai...
```

### Deploy (on merge to main) — `.github/workflows/deploy.yml`

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Build and push images
        run: |
          docker build -t ghcr.io/${{ github.repository }}/api:latest -f apps/api/Dockerfile .
          docker push ghcr.io/${{ github.repository }}/api:latest
          # repeat for web, ai...
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /srv/${{ github.event.repository.name }}
            docker compose -f docker-compose.yml -f docker-compose.prod.yml pull
            docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d --remove-orphans
            docker compose exec -T api npx prisma migrate deploy
            docker image prune -af
```

## Dockerfiles

### NestJS (multi-stage)

```dockerfile
# apps/api/Dockerfile
FROM node:24-alpine AS builder
WORKDIR /app
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY apps/api/package.json apps/api/
COPY packages/shared/package.json packages/shared/
RUN corepack enable && pnpm install --frozen-lockfile
COPY . .
RUN pnpm --filter=@my-product/api build

FROM node:24-alpine AS runner
WORKDIR /app
COPY --from=builder /app/apps/api/dist ./dist
COPY --from=builder /app/apps/api/node_modules ./node_modules
CMD ["node", "dist/main.js"]
```

### FastAPI

```dockerfile
# services/ai/Dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY services/ai/pyproject.toml services/ai/uv.lock ./
RUN pip install uv && uv sync --frozen --no-dev
COPY services/ai/src ./src
CMD ["uv", "run", "uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Next.js

```dockerfile
# apps/web/Dockerfile
FROM node:24-alpine AS builder
WORKDIR /app
# ... similar to NestJS but with next build output: standalone
RUN pnpm --filter=@my-product/web build

FROM node:24-alpine AS runner
COPY --from=builder /app/apps/web/.next/standalone ./
COPY --from=builder /app/apps/web/.next/static ./.next/static
CMD ["node", "server.js"]
```

## Environment variables in CI

All secrets flow through GitHub Environment Secrets or Repository Secrets:
- `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY` — for deploy SSH.
- `DB_USER`, `DB_PASSWORD`, etc. — injected at deploy time via `.env` on VPS,
  NOT hardcoded in workflow files.
- `.env` file on the VPS is created manually from the password manager.

## Reusable workflow (org-wide)

The `.github` repo can host a reusable CI template at
`.github/workflows/ci-template.yml` that product repos reference:

```yaml
# In a product repo's .github/workflows/ci.yml
jobs:
  ci:
    uses: The-Resonance-Team/.github/.github/workflows/ci-template.yml@main
    secrets: inherit
```

This avoids copying the CI boilerplate into every product repo.

## Rollback

```bash
# On the VPS:
docker compose down
# Checkout the previous image tag
IMAGE_TAG=<previous-sha> docker compose up -d
```

Git keeps the history; Docker images are tagged by SHA for this reason.
The deploy script can be extended to accept a `--rollback` flag.
