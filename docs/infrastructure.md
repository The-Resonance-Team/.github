# Infrastructure

Self-hosted Docker on VPS. What it is, why, and how to run it.

## Philosophy

We run our own servers. Paying a cloud markup for every compute second —
especially for AI workloads — isn't the right default for a small, versatile
team. We trade ops discipline for cost control and full-stack understanding.

## The stack

```
Internet → Traefik/Caddy (TLS) → Docker Compose network
                                      ├── nestjs-api
                                      ├── fastapi-ai
                                      ├── postgres
                                      ├── qdrant
                                      ├── redis (optional)
                                      └── observability (Prometheus, Loki, Grafana, GlitchTip)
```

## VPS provider

Choose per project. Recommended options (Singapore region preferred for
Vietnam latency):

| Provider | Strengths |
|---|---|
| Hetzner | Best price:performance; Singapore region via cloud arm |
| Vultr | Many Asia regions, simple |
| Contabo | Very cheap, big specs; mixed reliability anecdotes |
| FPT Cloud / Viettel | Local VN; latency + data compliance |

> **Default:** Hetzner, Singapore region. ARM instances (CAX) for web/API;
> CX for GPU if available. Revisit per product.

## Docker Compose structure

Each VPS runs one or more Compose stacks. A single-product VPS layout:

```
/srv/app/
├── docker-compose.yml           ← all services
├── docker-compose.prod.yml      ← overrides (replicas, resource limits)
├── .env                         ← NOT committed (synced from password manager)
├── .env.example                 ← committed, documents all required vars
├── traefik/
│   └── traefik.yml              ← static config
└── backups/
    └── pg_backup.sh             ← cron: pg_dump → Borg/S3
```

## Reverse proxy + TLS

Traefik or Caddy. Both do automatic Let's Encrypt TLS with no manual
certificate management.

Traefik (preferred, feature-richer):
```yaml
# traefik/traefik.yml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
  websecure:
    address: ":443"

certificatesResolvers:
  letsencrypt:
    acme:
      email: ops@resonance.team
      storage: /letsencrypt/acme.json
      httpChallenge:
        entryPoint: web

providers:
  docker:
    exposedByDefault: false
```

Each service gets TLS via Docker labels in the Compose file:
```yaml
services:
  api:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.myproduct.com`)"
      - "traefik.http.routers.api.tls.certResolver=letsencrypt"
```

## PostgreSQL

Run Postgres as a Compose service on the same VPS for single-server setups.
For higher availability: managed Postgres (Neon, Supabase, or a separate
dedicated VPS).

```yaml
services:
  postgres:
    image: postgres:17-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    restart: unless-stopped

volumes:
  pgdata:
```

**Backups:** a cron script running `pg_dump` + Borg or rclone to S3.

## Qdrant (vector DB)

```yaml
services:
  qdrant:
    image: qdrant/qdrant:latest
    volumes:
      - qdrant_data:/qdrant/storage
    environment:
      QDRANT__SERVICE__API_KEY: ${QDRANT_API_KEY}
    restart: unless-stopped

volumes:
  qdrant_data:
```

## GlitchTip (error tracking)

```yaml
services:
  glitchtip:
    image: glitchtip/glitchtip:latest
    environment:
      DATABASE_URL: postgres://...   # or sqlite for simplicity
      SECRET_KEY: ${GLITCHTIP_SECRET}
    labels:
      - "traefik.http.routers.errors.rule=Host(`errors.myproduct.com`)"
```

Point NestJS/FastAPI SDKs to `errors.myproduct.com`. GlitchTip SDKs are
Sentry-compatible (`@sentry/node`, `sentry-sdk` for Python).

## Grafana stack

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  loki:
    image: grafana/loki:latest

  grafana:
    image: grafana/grafana:latest
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
    labels:
      - "traefik.http.routers.dash.rule=Host(`dash.myproduct.com`)"
```

## Deploy flow

```
Developer pushes to main
  → GitHub Actions workflow runs lint/test/build
    → Builds Docker images for NestJS + FastAPI
    → Pushes images to ghcr.io
    → SSH into VPS
      → docker compose pull
      → docker compose up -d
      → docker compose exec api npx prisma migrate deploy (if needed)
```

## Security checklist

- [ ] SSH: key-only, non-standard port, no root login.
- [ ] Firewall: ports 80 and 443 only to the internet; Docker ports bound to
  `127.0.0.1` or internal network.
- [ ] TLS: Let's Encrypt auto-renew (Traefik/Caddy handles this).
- [ ] Docker: daemon never exposes a TCP port; Compose uses SSH.
- [ ] Updates: `unattended-upgrades` for OS; Watchtower or Renovate for
  container images (cautious — auto-updates can break prod).
- [ ] .env: never committed. Real values in a shared password manager.
  See [security.md](security.md).

## Scaling path

Docker Compose on a single VPS handles a lot. When a product outgrows it:

1. **Split services across VPSes:** DB on its own instance, AI on a GPU
   instance, web/API on compute.
2. **Adopt Swarm:** Docker Swarm for multi-node Compose without k8s.
3. **Consider k3s:** if the team is comfortable with Kubernetes and needs
   autoscaling.

None of these are the default — start with Compose on one VPS and upgrade
only when metrics prove you need it.
