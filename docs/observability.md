# Observability

How we know what our services are doing. Self-hosted, no SaaS dependency.

## The stack

| Signal | Tool | Port (internal) |
|---|---|---|
| Metrics | Prometheus | 9090 |
| Logs | Loki (aggregation) + structured JSON | 3100 |
| Dashboards + Alerts | Grafana | 3001 |
| Error tracking | GlitchTip | 8001 |

All are Docker Compose services on the same VPS (or a dedicated monitoring
VPS for larger setups).

## Metrics — Prometheus

Each service exposes a `/metrics` endpoint with its own Prometheus exporter:

| Service | Exporter |
|---|---|
| NestJS | `prom-client` (npm) |
| FastAPI | `prometheus-fastapi-instrumentator` |
| PostgreSQL | `prometheus-postgres-exporter` (sidecar) |
| Qdrant | built-in `/metrics` endpoint |
| Traefik | built-in `/metrics` endpoint |

### NestJS setup

```typescript
import { PrometheusExporter } from '@willsoto/nestjs-prometheus';

@Module({
  imports: [PrometheusExporter.forRoot({ defaultMetricsEnabled: true })],
})
export class MetricsModule {}
```

### FastAPI setup

```python
from prometheus_fastapi_instrumentator import Instrumentator

app = FastAPI()
Instrumentator().instrument(app).expose(app)
```

### Prometheus config

```yaml
# prometheus.yml
scrape_configs:
  - job_name: nestjs
    static_configs:
      - targets: ["nestjs-api:3000"]
        labels: { service: "api" }
  - job_name: fastapi-ai
    static_configs:
      - targets: ["fastapi-ai:8000"]
        labels: { service: "ai" }
```

### Grafana dashboards

Provision dashboards as code (JSON/YAML in `grafana/dashboards/`).
Default per-service dashboards:
- API: request rate, latency p50/p95/p99, error rate by route, active
  connections.
- AI: token count by provider, prompt latency, embedding latency, Qdrant
  query count.
- DB: connections, query latency, cache hit ratio, WAL lag.
- Host: CPU, memory, disk, network (node_exporter).

## Logs — Loki + structured JSON

### In-code logging

NestJS (Pino):
```typescript
import { Logger } from 'nestjs-pino';
// Config: pinoHttp with JSON output to stdout.
// Docker Compose ships stdout to Loki via the loki-docker-driver.
```

FastAPI (loguru):
```python
from loguru import logger
logger.add(sys.stdout, format="{time} {level} {message}", serialize=True)
```

### Loki + Promtail

Promtail tails Docker container logs and ships to Loki:

```yaml
# promtail config
scrape_configs:
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
    relabel_configs:
      - source_labels: [__meta_docker_container_name]
        target_label: container
```

Loki is the log store; Grafana queries it. All logs stay on the VPS.

## Error tracking — GlitchTip

GlitchTip is a self-hosted Sentry-compatible error tracker. SDKs are the
standard Sentry SDKs pointing at the GlitchTip instance.

### NestJS setup

```typescript
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.GLITCHTIP_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 0.1,
});
```

### FastAPI setup

```python
import sentry_sdk
sentry_sdk.init(dsn=os.environ["GLITCHTIP_DSN"], environment=os.environ["ENV"])
```

### GitHub integration

GlitchTip (like Sentry) can create GitHub issues for new errors. Enable
the integration per project.

## Alerts

Grafana Alertmanager — define alert rules in code, provisioned with the
dashboard config:

```yaml
# Example alert rule
groups:
  - name: api
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "API error rate > 5% for 5 minutes"
```

Alert channels: email (SMTP from the VPS), Discord webhook, or Telegram
(for VN team communication).

## Health checks

Every service has a `/health` endpoint:

```
GET /health
→ 200 { "status": "ok", "uptime": 12345, "dependencies": { "db": "ok", "qdrant": "ok" } }
```

Traefik uses these for load balancing health checks.

## On-call

- Grafana alerts → Discord/Telegram notification.
- On-call rotation defined in the team's communication channel (not
  automated in v1 — keep it simple).
- Pager duty optional unless the team grows past informal rotation.

## Minimum for a new app

At launch, a new product MUST have:
- [ ] Prometheus metrics on API + AI service.
- [ ] Structured JSON logs shipping to Loki.
- [ ] GlitchTip configured with `@sentry/node` + `sentry_sdk`.
- [ ] Grafana dashboard with at least the "API health" panel.
- [ ] One alert: API error rate > threshold.

Everything else is added as the product matures.
