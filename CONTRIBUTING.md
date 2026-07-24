# Contributing

This repo is the team's standards playbook — proposals to change the stack or
workflow start here.

## Before proposing a change

1. Read [docs/tech-stack.md](docs/tech-stack.md) — current defaults.
2. Read [docs/architecture.md](docs/architecture.md) — system shape and
   constraints.
3. Check [docs/adr/](docs/adr/) — was this already decided? What was the
   reasoning?
4. Read [docs/developer-loop.md](docs/developer-loop.md) and the project
   overlay template before changing workflow or project structure.

## Proposal process

1. Open an issue describing the change and the problem it solves. Use the
   **config** issue template for convention changes.
2. Discussion on the issue. If it looks like a go:
3. Draft the change as a PR touching the relevant `docs/*.md` and adding a new
   ADR in `docs/adr/`.
4. At least one review from a core team member required.
5. Merge; the ADR is now the source of truth.

## Canonical developer loop

The organization-wide engineering loop lives in
[docs/developer-loop.md](docs/developer-loop.md). Projects adopt it with
[the project overlay template](docs/templates/project-developer-loop.md) and
record continuation state with [the handoff template](docs/templates/developer-handoff.md).

The current user request, safety constraints, live repository, and active
handoff are the authority order. Session transcripts and external global files
are historical evidence only.
