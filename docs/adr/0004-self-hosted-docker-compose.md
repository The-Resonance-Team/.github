# ADR 0004: Self-hosted Docker Compose on VPS

- **Date:** 2026-06-24
- **Status:** accepted

## Context

The team needs a deployment target for Next.js, NestJS, and FastAPI
services. Cloud platforms (Vercel, Heroku, Railway) trade cost and control
for convenience. The team wants full control and predictable pricing.

## Decision

All services run as Docker containers on a **self-managed VPS** (or own
server), orchestrated with **Docker Compose**. A reverse proxy (Traefik or
Caddy) handles TLS termination via Let's Encrypt.

The default VPS provider is **Hetzner** (Singapore region) for its
price-to-performance ratio. The default region is **Singapore** for low
latency to Vietnam.

## Consequences

- **Positive:** Predictable, flat-cost infrastructure. No per-request or
  per-compute-second billing surprises.
- **Positive:** Full control over the environment — run any binary, any
  version, any database extension.
- **Positive:** Team builds deep ops knowledge (Docker, TLS, networking,
  backups) — valuable for long-term platform ownership.
- **Negative:** Ops burden: the team is responsible for uptime, backups,
  security patches, and incident response. No PaaS safety net.
- **Negative:** Scaling requires manual intervention (provision a larger
  VPS, add Swarm or k3s). Acceptable until a product outgrows a single box.
