# ADR 0001: TypeScript for web and Zalo surfaces

- **Date:** 2026-06-24
- **Status:** accepted

## Context

The Resonance Team ships AI applications across multiple surfaces: web,
mobile, and Zalo Mini App. A language choice for the web and Zalo surfaces
determines code sharing, hiring, and toolchain investment.

## Decision

TypeScript is the standard language for the **web** (Next.js), **NestJS API**,
and **Zalo Mini App** surfaces. All three are JS-runtime-capable; sharing one
language enables a shared types package (`@resonance/shared`) and a single
monorepo toolchain (Turborepo + pnpm).

Non-TS surfaces (Flutter, Python) are separate by necessity, not by choice.

## Consequences

- **Positive:** Web, API, and Zalo share DTOs and API client types via
  `@resonance/shared` — no drift between client and server contracts.
- **Positive:** One lint config, one test framework (vitest), one hiring
  profile for the majority of backend + frontend code.
- **Negative:** Flutter (Dart) and AI service (Python) cannot consume TS
  types directly; contract artifacts must be generated or kept in sync
  manually.
