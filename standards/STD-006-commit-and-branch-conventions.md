# STD-006 — Commit & Branch Conventions

Normative for all five repositories. Strategy context:
[architecture/008](../architecture/008-git-strategy.md),
[ADR-009](../adr/ADR-009-versioning.md).

## Branches

| Prefix | For |
|---|---|
| `feat/<kebab-topic>` | New service, capability, workflow |
| `fix/<kebab-topic>` | Bug/config fixes |
| `docs/<kebab-topic>` | Documentation only |
| `chore/<kebab-topic>` | Pins bumps, maintenance, tooling |

Branches MUST be short-lived (days). `main` is protected: PR + green CI
required, squash merge, no force pushes.

## Commits — Conventional Commits

```text
<type>(<optional scope>): <imperative summary ≤ 72 chars>
```

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `ci`.
Scope = stack or area: `feat(vaultwarden): add stack`,
`chore(monitoring): bump grafana to 11.4.0`.

Breaking a cross-repo contract
([architecture/002](../architecture/002-repository-model.md)) MUST use
`!` and a `BREAKING CHANGE:` footer — this drives the major version bump
via git-cliff.

## Pull requests

The PR description MUST answer, briefly:

1. **What changed** — one paragraph.
2. **Which stacks redeploy** — so the deploy blast radius is stated
   before merge.
3. **Rollback plan** — usually "revert this PR"; say so explicitly, or
   explain when it isn't (data migrations!).

Self-merge after green CI is the process, honestly
([ADR-009](../adr/ADR-009-versioning.md)); the written answers above are
the review.

## Tags

`vMAJOR.MINOR.PATCH`, annotated, never moved or deleted. Tagging is the
closing step of each monthly maintenance window
([RB-006](../runbooks/RB-006-update-services.md)).
