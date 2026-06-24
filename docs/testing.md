# Testing

Per-language test framework defaults and expectations.

## Principles

- **Test at the service boundary.** Unit tests internal logic; integration
  tests verify the contract (API routes, DB queries, LLM calls mock the
  provider).
- **No flaky tests in CI.** Flaky tests must be fixed or quarantined
  (`.skip` + issue). A failing test means a failing build.
- **E2E covers critical flows only.** Login, core CRUD, payment if
  applicable. Don't e2e every edge case.

## TypeScript (web + NestJS API + Zalo)

| Concern | Tool |
|---|---|
| Test runner | vitest |
| Assertions | vitest `expect` |
| Mocking | vitest mocks + `msw` (API mocking) |
| E2E | Playwright |

### vitest config (shared workspace)

```typescript
// tools/vitest-config/vitest.base.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov'],
      include: ['src/**'],
    },
  },
});
```

Each app extends: `vitest.config.ts` extending the base config.

### Playwright for E2E

```bash
pnpm exec playwright test
```

Test the deployed app (against staging or local Compose). Focus on:
- Auth flow (login → protected route → logout).
- Core product action (create → read → update → delete).
- Error states (500 page, 404, rate limit).

## Python (FastAPI AI service)

| Concern | Tool |
|---|---|
| Test runner | pytest |
| Async support | pytest-asyncio |
| HTTP testing | httpx + `TestClient` (FastAPI) |
| Mocking | `unittest.mock` or `pytest-mock` |
| Lint | ruff |

### Structure

```
services/ai/
├── tests/
│   ├── conftest.py         ← shared fixtures (TestClient, mock OpenAI)
│   ├── test_routes.py      ← endpoint tests
│   ├── test_providers.py   ← provider routing logic
│   └── test_rag.py         ← RAG pipeline tests
```

### Fixture pattern

```python
# tests/conftest.py
import pytest
from httpx import AsyncClient, ASGITransport
from src.main import app

@pytest.fixture
async def client():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac

@pytest.fixture
def mock_openai(mocker):
    return mocker.patch("src.providers.openai_client.chat.completions.create")
```

## Dart (Flutter mobile)

| Concern | Tool |
|---|---|
| Unit test | `flutter test` |
| Widget test | `flutter test` (widget test utils) |
| Integration | `integration_test` package |
| Mocking | `mocktail` or `mockito` |

## Test in CI

All test suites run in parallel per PR:

```yaml
# turbo.json
{
  "tasks": {
    "test": { "dependsOn": ["^build"], "outputs": ["coverage/**"] }
  }
}
```

Running `turbo test` runs vitest + pytest (via shim) + lint concurrently.

## Coverage expectations

| Layer | Target | Notes |
|---|---|---|
| Business logic (NestJS services) | 80%+ | Critical for correctness |
| AI provider routing (FastAPI) | 80%+ | Provider swaps must not break |
| UI components (Next.js) | 60%+ | Focus on data display + forms |
| E2E flows | 100% of critical paths | Not by coverage %, by feature |
| Utils / helpers | 90%+ | Cheap to test, high leverage |

These are **aspirational** — new projects start lower and grow into them.
