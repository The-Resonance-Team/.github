# Repositories

How code is organized across repos. New applications follow this pattern.

## Default: Turborepo monorepo

A product = one Turborepo monorepo containing the TS family:

```
my-product/
├── turbo.json
├── pnpm-workspace.yaml
├── package.json                ← root (workspaces)
├── apps/
│   ├── web/                    ← Next.js (App Router, TS)
│   ├── api/                    ← NestJS (TS)
│   └── zalo/                   ← ZMP SDK (optional, VN-market)
├── packages/
│   └── shared/                 ← @resonance/shared: types, DTOs, API client
├── services/
│   └── ai/                     ← FastAPI Python (see shim trick below)
│       ├── package.json        ← thin shim for turbo
│       ├── pyproject.toml
│       ├── src/
│       └── tests/
└── tools/
    └── eslint-config/          ← shared ESLint config
```

## Python in the monorepo — the `package.json` shim

The FastAPI AI service lives in the monorepo but Python tooling doesn't
speak pnpm. A thin `package.json` bridges them:

```json
{
  "name": "@my-product/ai",
  "private": true,
  "scripts": {
    "dev": "uv run uvicorn src.main:app --reload",
    "test": "uv run pytest",
    "lint": "uv run ruff check .",
    "typecheck": "uv run pyright ."
  }
}
```

In `turbo.json`:
```json
{
  "tasks": {
    "dev": { "cache": false, "persistent": true },
    "test": { "dependsOn": ["^build"] },
    "lint": {},
    "typecheck": {}
  }
}
```

Turbo runs `pnpm test` → pnpm delegates to the AI package → `uv run pytest`.
Turbo caches the output hash, parallelizes, and respects `dependsOn`.

## Shared types package

`packages/shared/` is the single source of truth for all API contracts:

```
packages/shared/
├── package.json      ← @resonance/shared (published to npm)
├── src/
│   ├── dto/          ← request/response DTOs (Zod schemas + TS types)
│   ├── client/       ← typed API client (fetch wrapper, generated or hand-written)
│   └── const/        ← shared constants (error codes, enums)
├── tsconfig.json
└── vitest.config.ts
```

**Web / NestJS** consume `@resonance/shared` as a workspace dependency
(`"@resonance/shared": "workspace:*"`).

**Flutter** cannot directly consume the TS package. Two options:
- Publish `@resonance/shared` as an npm package, generate Dart types via
  `openapi-generator` from the NestJS swagger spec.
- Manually keep Dart models in sync (pragma: not worth automating for
  a small team; keep shared DTOs simple).

## Flutter repo (separate)

Flutter lives in its own repo because Dart tooling is orthogonal to
Turborepo and the monorepo's value diminishes:

```
my-product-mobile/
├── pubspec.yaml
├── lib/
├── test/
└── ios/ + android/
```

If the team wants Flutter in the monorepo: use the same package.json shim
trick (a `mobile/` workspace with `package.json` calling `flutter` commands).
Not the default; the playbook recommends separate.

## Zalo Mini App (optional)

When a product ships the Zalo surface, add it as:

```
apps/zalo/
├── package.json
├── src/
│   ├── app.js          ← ZMP entry point
│   └── ...
└── README.md
```

ZMP SDK is HTML/JS — lives naturally in the pnpm workspace. Share types
from `@resonance/shared` directly.

## Starting a new product

See [new-app-checklist.md](new-app-checklist.md) for the step-by-step
bootstrap recipe.
