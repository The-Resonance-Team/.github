# Project Developer Loop Overlay

Copy this file to `docs/developer-loop/PROJECT.md` in each project and fill in
the project-specific facts. Keep the loop itself consistent with
[`docs/developer-loop.md`](../developer-loop.md).

## Project identity

- Project:
- Repository:
- Local root:
- Default branch:
- Primary owner:
- Runtime/languages:
- Application surfaces:

## Anchor defaults

- Allowed top-level paths:
- Read-only paths or generated paths:
- Default mutation mode:
- Required handoff location:
- Required issue or PR reference:

## Local code style and structure

- Formatter command:
- Lint command:
- Typecheck command:
- Focused test command:
- Integration or database test command:
- E2E, browser, device, or runtime command:
- Naming/import conventions:
- Module boundaries and file-placement rules:
- Test placement rules:

Use existing repository tooling and nearby stable modules as the authority.
Record exceptions here with a reason and an expiry or removal condition.

## Domain and safety overlay

- Canonical domain context:
- Important terms:
- Roles and authorization boundaries:
- Sensitive data or privacy boundaries:
- Allowed agent actions:
- Human approval gates:
- Forbidden or out-of-scope actions:

## Proof policy

| Change risk | Required proof | Evidence location |
| --- | --- | --- |
| Baseline |  |  |
| Service/data |  |  |
| UI/device |  |  |
| Auth/mutation |  |  |
| Production/safety |  |  |

Never mark a lower proof tier as complete feature verification.

## Delivery policy

- Commit format:
- Branch format:
- Review requirement:
- Merge authority:
- Deployment authority:
- Production mutation authority:
- Cleanup or rollback procedure:

## Adoption checklist

- [ ] Project identity and anchor defaults are complete.
- [ ] Local style and structure commands are verified against the live tree.
- [ ] Domain and safety boundaries are explicit.
- [ ] Proof commands are executable and scoped.
- [ ] Delivery and mutation authority are explicit.
- [ ] Handoff location exists.
- [ ] This overlay does not copy the full organization standard.
