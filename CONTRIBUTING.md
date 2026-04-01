# Contributing

## Default workflow

1. Start from a tracked issue or create one first.
2. Create a branch from `main` using one of these prefixes:
   - `feature/<ticket>-<slug>`
   - `fix/<ticket>-<slug>`
   - `release/<version>`
   - `hotfix/<version>`
3. Open a pull request early.
4. Request review from the relevant maintainers.
5. Merge with squash once checks and review are complete.

## Pull request expectations

- Explain the change clearly.
- Link the related issue or task.
- Include testing evidence.
- Add rollback notes for risky changes.
- Keep scope focused.

## Review policy

- Non-trivial changes should go through pull requests.
- Only org owners and designated maintainers should merge into `main`.
- Direct pushes to `main` are reserved for urgent fixes or explicit owner approval.

## Commit guidance

Use short, descriptive commits. Prefer one concern per commit.
