# ADR 0008: Turborepo monorepo with Python shim

- **Date:** 2026-06-24
- **Status:** accepted

## Context

A product spans multiple surfaces (web, API, AI, optionally Zalo) and
multiple languages (TypeScript, Python). The team wants a single repo that
contains all of a product's code but Turborepo is TypeScript-native.

## Decision

Use a **Turborepo monorepo** (pnpm workspaces) as the default structure
for a product. All TypeScript apps (`apps/web`, `apps/api`, `apps/zalo`,
`packages/shared`) live natively in the workspace.

The **FastAPI Python service** lives in the monorepo as a `services/ai/`
directory with a thin `package.json` shim whose scripts shell out to
`uv` commands (`uv run pytest`, `uv run uvicorn`, etc.). Turbo runs and
caches these via the same task pipeline as TS tasks.

## Consequences

- **Positive:** One repo per product — no cross-repo PR coordination for
  changes that touch API + AI + web together.
- **Positive:** Turbo's caching and parallel execution work for Python
  tasks through the shim, saving CI minutes on Python lint/test/build.
- **Positive:** Developers run `turbo dev` and get web + API + AI all
  starting together.
- **Negative:** The `package.json` shim is non-standard — new team members
  must learn the indirection. Template repos and this ADR document it.
- **Negative:** Flutter (Dart) does not benefit from the monorepo and
  lives in a separate repo by default. If Flutter needs monorepo
  inclusion, the same shim pattern applies but is not the default.
