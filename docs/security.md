# Security

Security defaults and mandatory rules for Resonance Team applications.

## Secrets management

### `.env` rules (mandatory)

1. `.env` files are **never committed**. This is enforced: `.env` and
   `*.env` are in `.gitignore` at the repo root.
2. Every repo has a committed `.env.example` that documents all required
   environment variables with placeholder values. No real secrets.
3. Real `.env` values live in the team's shared password manager
   (1Password / Bitwarden). They are synced manually to each VPS.
4. On deploy: the `.env` file is already on the VPS (created once, updated
   when variables change). CI does NOT transmit secrets over SSH — it
   reads them from the existing VPS `.env`.
5. After a team member leaves, rotate all secrets they had access to.

### `git-crypt` rules

Some secrets legitimately live in the repo (CI workflow templates,
pre-commit hook configs that reference internal tooling). Lock these with
`git-crypt`:

```bash
git-crypt init
git-crypt add-gpg-user <key-id>
# Files matching patterns in .gitattributes get encrypted automatically
```

> **Rule:** `git-crypt` is for *repository-level* secrets that must travel
> with the code. App secrets (API keys, DB passwords, JWT secrets) go in
> `.env` and the password manager. Never mix.

## Authentication

- **NestJS + Passport.js** handles all user authentication.
- Default: stateless JWT (access + refresh tokens). Sessions used only when
  a product specifically needs them.
- OAuth2 providers (Google, Facebook, Zalo) configured per product.
  Passport strategies for each.
- No auth logic in the FastAPI AI service — it trusts the NestJS caller on
  the internal Docker network.
- Rate limiting on the auth endpoints (express-rate-limit or NestJS
  ThrottlerModule).

## Network

- **Reverse proxy:** only Traefik/Caddy is exposed to the internet (ports
  80/443). All other services bind to `127.0.0.1` or the internal Docker
  network.
- **SSH:** key-only authentication, no root login, non-standard port.
- **Firewall (ufw):** allow 80/tcp, 443/tcp, SSH port. Deny all else.
- **Inter-service:** NestJS → FastAPI over the internal Docker network
  (no TLS needed, but a shared secret header for basic trust).

## Database

- Postgres not exposed to the internet. Connections are from the Docker
  network only.
- Application user has minimum required privileges (owns its schema but
  no SUPERUSER / CREATEDB).
- Migrations run via CI (Prisma/Drizzle migrate) — never manual DDL in prod.
- Connection pooling: PgBouncer or built-in ORM pool. Connection strings
  include `?sslmode=disable` (internal network).

## Dependencies

- Regular `pnpm audit` / `pip audit` runs in CI.
- Renovate bot configured for automated dependency PRs. Merge patch
  updates aggressively; review minor/major.
- Docker base images pinned to digest SHAs in CI builds (`node:24-alpine@sha256:...`).

## Secrets rotation schedule

| Secret type | Rotation | Method |
|---|---|---|
| DB passwords | 90 days | Manual; update `.env`, restart service |
| JWT signing keys | 180 days | Generate new key pair; old tokens expire naturally |
| API keys (external) | Per provider policy | Manual |
| SSH keys | Per year or on personnel change | Replace in VPS `authorized_keys` |
| git-crypt keys | On personnel change | Re-encrypt with remaining GPG keys |

## Incident response

When a secret leaks:
1. Rotate the secret immediately.
2. Revoke any session tokens issued since the leak.
3. Check access logs for the window between leak and rotation.
4. Post-mortem: add a rule or automation that would have prevented it.

For the full policy including incident templates, see `SECURITY.md` at
the repo root.
