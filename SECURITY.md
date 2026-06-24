# Security

## Reporting a vulnerability

Do not create public issues for security vulnerabilities.
Email **[security@resonance.team]** (pending setup).

## Secrets policy

- `.env` files are **never committed**. A `.env.example` with placeholder keys
  documents the schema. Real values live in a shared password manager
  (1Password / Bitwarden) and are copied to each environment manually.
- `git-crypt` locks any committed secret templates (see `.gitattributes`).
  Rotate all secrets when a team member loses access.
- Rotation: credentials expire by default after 90 days; automate where
  possible (GitHub token rotation, DB password rotation).

## Infrastructure

- All services behind a reverse proxy (Traefik / Caddy) with automatic TLS via
  Let's Encrypt.
- Firewall: only ports 80/443 open to the internet; SSH on a non-standard port
  or VPN-only.
- Docker daemon not exposed — Compose over SSH.

For the full policy see [docs/security.md](docs/security.md).
