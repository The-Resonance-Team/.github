# ADR 0006: NestJS + Passport.js for authentication

- **Date:** 2026-06-24
- **Status:** accepted

## Context

Every product needs authentication. The team considered managed auth
(Supabase Auth, Clerk) versus self-hosted (Keycloak, Authentik) versus
framework-native (NestJS + Passport.js).

## Decision

Authentication lives in the **NestJS API service** using the **Passport.js**
ecosystem. Default strategy: stateless JWT (access + refresh tokens).
OAuth2 providers (Google, Facebook, Zalo) are added via Passport strategies
per product.

No separate auth service. No managed auth provider.

## Consequences

- **Positive:** Auth is colocated with the API that enforces it — no
  network hop to an external identity provider for every request.
- **Positive:** Passport.js has strategies for every OAuth2 provider the
  team is likely to use, including Zalo (custom strategy).
- **Positive:** Fits the team's self-host ethos — no external auth
  dependency, no per-user auth billing.
- **Negative:** The team owns auth security (JWT rotation, token
  revocation, session management, brute-force protection). Auth bugs are
  among the most severe — there is no vendor safety net.
- **Negative:** No built-in admin UI for user management. The team must
  build or script user management operations.
