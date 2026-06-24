# Contributing

This repo is the team's standards playbook — proposals to change the stack or
workflow start here.

## Before proposing a change

1. Read [docs/tech-stack.md](docs/tech-stack.md) — current defaults.
2. Read [docs/architecture.md](docs/architecture.md) — system shape and
   constraints.
3. Check [docs/adr/](docs/adr/) — was this already decided? What was the
   reasoning?

## Proposal process

1. Open an issue describing the change and the problem it solves. Use the
   **config** issue template for convention changes.
2. Discussion on the issue. If it looks like a go:
3. Draft the change as a PR touching the relevant `docs/*.md` and adding a new
   ADR in `docs/adr/`.
4. At least one review from a core team member required.
5. Merge; the ADR is now the source of truth.

## Reuse

The agent workflow layer (triage labels, issue tracker conventions, domain
documents) lives in the dotfiles repo — see `~/workspace/dotfiles/docs/agents/`.
This playbook links those, it doesn't duplicate them.
