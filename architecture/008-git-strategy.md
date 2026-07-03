# 008 — Git Strategy

Rationale for the model: [ADR-009](../adr/ADR-009-versioning.md) (solo
maintainer, continuous deploy). Normative conventions:
[STD-006](../standards/STD-006-commit-and-branch-conventions.md).

## Model: trunk-based

- `main` is always deployable and **is** what is deployed.
- No `develop` branch, no gitflow, no release branches — for a solo
  maintainer with continuous deployment they are pure ceremony.
- Short-lived branches (days, not weeks): `feat/<topic>`, `fix/<topic>`,
  `docs/<topic>`, `chore/<topic>`.

## Branch protection (all five repos)

| Setting | Value |
|---|---|
| Require pull request before merging | ✅ (self-merge allowed after CI) |
| Require status checks (`ci`) | ✅ |
| Force pushes / deletions on `main` | ❌ blocked |
| Merge style | Squash (one commit per change, clean history) |

The PR is not review theater: the template asks *what changed, which
stacks redeploy, what is the rollback plan* — writing it catches mistakes
even self-reviewed.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/):
`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `ci:`. These feed
changelog generation (git-cliff) at release time
([010-release-management.md](010-release-management.md)).

## What lives in Git — and what never does

| In Git | Never in Git |
|---|---|
| Compose files, configs, scripts, workflows, docs, dashboards (JSON), alert rules, dnsmasq rules | Secrets and `.env` files ([ADR-008](../adr/ADR-008-secrets-management.md)) |
| `.env.example` with every variable, dummy values | Runtime data (`/srv/homelab/data`) |
| Diagram sources + exports | TLS private keys |

Enforced by gitleaks in every repo's CI **and** as a pre-commit hook.

## Immutable history

- ADRs: never edited after acceptance — superseded by a new ADR.
- `CHANGELOG.md`: append-only.
- Tags: never moved or deleted.
