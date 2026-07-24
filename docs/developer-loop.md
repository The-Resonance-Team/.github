# Developer Loop

The Developer Loop is the source of truth for how Resonance Team projects move
from a bounded request to verified delivery. Every project uses the same loop;
each project supplies only its local code, structure, domain, and safety
overlay.

This document governs engineering work across repositories. It does not replace
a project's domain context, architecture decisions, or application-specific
runbooks.

## Authority order

When evidence conflicts, use this order:

1. The current user request and its safety constraints.
2. The current repository: root, branch, worktree, code, tests, configuration,
   and observable runtime behavior.
3. The active handoff, as intended continuation context.

Claude/OpenCode transcripts, session titles, generated summaries, and external
global instruction files are historical evidence only. They must not override
the current repository or silently expand scope.

If a handoff disagrees with the repository, stop before editing, record the
drift, and re-establish the intended scope.

## Standard loop

```text
Anchor → Model → Inspect → Change → Prove → Build/CI → Package/Handoff → Circle
```

### 1. Anchor

Create an anchor before exploration or editing. It records:

- repository root;
- branch and worktree;
- session ID and parent handoff;
- requested outcome;
- allowed files or modules;
- mutation mode: read-only, local-edit, draft-only, or real CRUD;
- dirty-worktree state.

The anchor is the first defense against cross-project and cross-session
contamination.

### 2. Model

Make the work precise before implementation:

- define the user-visible outcome;
- identify domain terms and boundaries;
- state acceptance criteria;
- identify safety, authorization, privacy, and mutation constraints;
- choose the proof tier below.

### 3. Inspect

Inspect the live repository and adjacent stable code before editing. Confirm the
current file structure, existing tests, configuration, dependencies, and
handoff assumptions. Do not infer current behavior from a transcript alone.

### 4. Change

Make the smallest change that satisfies the model. Add or update the focused
test with the implementation. Keep unrelated files and user changes outside
the task scope untouched.

### 5. Prove

Use the lowest proof tier that fully covers the risk; escalate when the change
touches a higher-risk boundary.

| Tier | Required proof |
| --- | --- |
| Baseline | Scoped diff, repository formatter, lint, typecheck, and focused tests |
| Service/data | Baseline plus integration, database, and contract checks |
| UI/device | Service/data as applicable plus real interaction and reload/persistence checks |
| Auth/mutation | Role-boundary checks, observable side effects, cleanup or cancellation, and redacted evidence |
| Production/safety | Explicit authorization, rollback or recovery path, runtime evidence, and residual-risk review |

Route smoke, HTTP 200, a green tool trace, or file presence is not full feature
verification.

### 6. Build/CI

Classify failures before retrying. A timeout or dependency outage is not a
product defect until the product behavior is isolated. Record the command,
environment, result, and whether the failure is reproducible.

### 7. Package/Handoff

Before delivery:

- audit the diff and staged paths;
- confirm branch and worktree state;
- compare the result with the requested outcome and active handoff;
- record proof and residual risk;
- commit, open a PR, merge, deploy, or perform production CRUD only when that
  authority is explicit.

Every interrupted, blocked, or continued loop requires a durable handoff.

### 8. Circle

The loop is iterative, not linear:

- provenance or scope failure → stop and clarify;
- product/code failure → Inspect or Change;
- verification failure → inspect the failed proof, then Change;
- environment/tool failure → bounded retry, then escalate;
- delivery failure → Package/Handoff;
- successful proof → deliver or close.

Never retry indefinitely and never restart from an unrelated session by
default.

## Failure taxonomy

Every failure gets one primary class and optional secondary tags.

| Class | Meaning | Default response |
| --- | --- | --- |
| `provenance` | Wrong project, branch, worktree, or session | Stop and re-anchor |
| `scope` | Unauthorized file, mutation, or external action | Stop and clarify authority |
| `product` | Implementation or behavior defect | Inspect and change |
| `verification` | Failed test, lint, typecheck, build, or runtime proof | Inspect the failed proof |
| `environment` | Database, network, dependency, timeout, or tool failure | Bounded retry and isolate |
| `delivery` | Git, CI, PR, deployment, or sync failure | Package and recover |

## Code style and file structure

The standard is repository-local, not a global style imposed across all
projects:

- use the current repository's formatter, linter, typechecker, and test
  configuration;
- follow the nearest stable neighboring module for naming, imports, errors,
  tests, and file placement;
- preserve existing module boundaries unless the work explicitly changes
  architecture;
- do not perform a broad cleanup to make unrelated files consistent;
- document a deliberate local exception in the project overlay or handoff.

## Delegation

Subagents have explicit ownership and report their repository, branch, scope,
evidence, and failures.

| Role | Default authority |
| --- | --- |
| Explorer | Read-only repository and context analysis |
| Domain/spec reviewer | Terms, scope, acceptance criteria |
| Standards reviewer | Style and file-structure compliance |
| Implementer | One bounded write scope |
| Verification reviewer | Proof execution and residual risk |
| Security/safety reviewer | Elevated-risk changes only |

One primary agent integrates results. Two agents must not edit the same files.

## Handoff contract

Use [`templates/developer-handoff.md`](templates/developer-handoff.md) for any
work that may continue across sessions. A handoff contains the anchor, outcome,
scope, decisions, changed files, exact proof, classified failures, residual
risk, and the next action. Do not put credentials, tokens, or unnecessary
personal data in a handoff.

Each project adopts this standard with
[`templates/project-developer-loop.md`](templates/project-developer-loop.md).

## Completion rule

Work is complete only when the requested outcome is implemented or explicitly
blocked, the appropriate proof tier is recorded, the final scope is audited,
and the next state is clear. Passing tests alone does not close the loop.
