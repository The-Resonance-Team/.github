# Workflow

Engineering workflow conventions for Resonance Team projects.

## Branching — GitHub Flow

```
main ─────────────────────────────────────────────────────
  │
  ├── feat/add-auth ── (PR) ── squash merge ──→ main
  │
  ├── fix/login-timeout ── (PR) ── squash merge ──→ main
  │
  └── chore/update-deps ── (PR) ── squash merge ──→ main
```

**Rules:**
- `main` is always deployable. Every merge to main triggers CI → deploy.
- Feature branches are short-lived (hours to a few days). If a branch lives
  longer than a week, break it into smaller PRs.
- Branch names use conventional commit prefixes: `feat/`, `fix/`, `chore/`,
  `docs/`, `refactor/`, `test/`.
- No direct commits to `main`.

## Commits — Conventional Commits

```
feat(api): add JWT refresh token rotation
fix(web): correct login redirect after session expiry
chore(deps): bump next from 15.2 to 15.3
docs(adr): add decision record for vector DB choice
refactor(ai): extract provider routing into dedicated module
test(api): add e2e for auth edge cases
```

**Format:** `<type>(<scope>): <description>`

| Type | When |
|---|---|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `chore` | Maintenance (deps, config, CI) |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes nor adds a feature |
| `test` | Adding or updating tests |
| `perf` | Performance improvement |

Scope is the affected workspace: `web`, `api`, `ai`, `shared`, `zalo`, `mobile`.

## Pull requests

### Template

```markdown
## Summary
<!-- 1-2 sentences describing the change -->

## Motivation
<!-- Why is this change needed? Link the issue. -->

## Checklist
- [ ] Tests pass
- [ ] Lint passes
- [ ] Type-check passes
- [ ] Conventional commit title
- [ ] Related ADR updated or added (if architectural)
```

### Review rules

- **One approval required** before merge.
- Reviewer checks: correctness, test coverage, ADR consistency, security
  (secrets in diff? new endpoints have auth guards?).
- Author merges their own PR after approval (self-merge, not a reviewer
  gate).
- Squash-merge: one conventional commit per PR on `main`. The PR title
  becomes the commit message.

## Issues

Issue templates in `.github/ISSUE_TEMPLATE/`:
- **Bug report** (`bug.yml`) — reproduction steps, expected vs actual,
  environment.
- **Feature request** (`feature.yml`) — problem statement, proposed
  solution, alternatives considered.
- **Config / convention change** (`config.yml`) — what standard to change,
  why, impact on existing products.

### Triage labels

Reuse the shared triage label vocabulary from the dotfiles:
`~/workspace/dotfiles/docs/agents/triage-labels.md`.

Five labels: `triage/needs-investigation`, `triage/accepted`,
`triage/blocked`, `triage/duplicate`, `triage/wont-fix`.

## Definition of done

A feature is "done" when:
- [ ] Tests pass (unit + integration, e2e where applicable).
- [ ] Lint and type-check pass.
- [ ] PR reviewed and approved.
- [ ] Merged to `main` and deployed to production.
- [ ] Relevant ADR created or updated.
- [ ] Observability: feature has metrics/logs if it's a new service path.

## Release cadence

No scheduled releases. **Continuous deployment** — every merge to `main`
deploys. This forces small, safe changes.

Exceptions (mobile): App Store review cycles require manual promotion.
Mobile merges to `main` still trigger a build but the deploy step (publish
to App Store / Play Store) is gated on a manual trigger or a release
tag (`v1.2.0`).

## Tools

| Concern | Tool |
|---|---|
| Lint (TS) | ESLint + Prettier (shared config in `packages/eslint-config`) |
| Lint (Python) | ruff |
| Format (TS) | Prettier |
| Format (Python) | ruff format |
| Pre-commit | husky + lint-staged (TS); pre-commit (Python) |
| Commit lint | commitlint (enforces Conventional Commits) |

## Reusing agent workflow docs

The agent workflow layer (issue tracker conventions, triage labels, domain
documents) is defined in `~/workspace/dotfiles/docs/agents/`. This playbook
**links** those; it does not redefine them.

- [Issue tracker conventions](https://github.com/xirothedev/dotfiles/blob/main/docs/agents/issue-tracker.md)
- [Triage labels](https://github.com/xirothedev/dotfiles/blob/main/docs/agents/triage-labels.md)
- [Domain documentation](https://github.com/xirothedev/dotfiles/blob/main/docs/agents/domain.md)
