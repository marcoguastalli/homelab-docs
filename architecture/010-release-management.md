# 010 — Release Management

Rationale: [ADR-009](../adr/ADR-009-versioning.md) (versioning) and
[ADR-010](../adr/ADR-010-release-strategy.md) (release strategy).

## Deployment ≠ release

- **Deployment** is continuous: every merge to `main` deploys.
- **A release (tag)** marks a *known-good platform state* for humans and
  disaster recovery. "Restore the platform as of
  `infrastructure v1.4.0 + services v2.1.3`" is a precise, testable
  statement.

## Versioning

| Aspect | Rule |
|---|---|
| Scheme | SemVer per repository, independent (`vMAJOR.MINOR.PATCH`) |
| Major | Breaking change to a cross-repo contract ([002](002-repository-model.md)) — e.g. renaming `net_proxy`, changing `deploy.sh` CLI |
| Minor | New service / stage / feature / workflow |
| Patch | Fixes, config tweaks, image pin bumps |

## Release procedure

1. Ensure `main` is green (nightly validation confirms continuously).
2. Tag: `git tag v1.4.0 && git push origin v1.4.0`.
3. `release.yml` generates the changelog (git-cliff, from conventional
   commits) and publishes a GitHub Release automatically.

Cadence: after each meaningful milestone and after each monthly
maintenance window ([011](011-operations-model.md)) — not time-boxed.

## Breaking changes across repos

A breaking change in a producer repo REQUIRES, in this order:

1. An ADR (or an update superseding one) recording the new contract.
2. A **major** version bump of the producer.
3. A migration note in each consumer repo's `CHANGELOG.md`.
4. Consumers updated in the same maintenance window.

## Rollback semantics

| Failure | Rollback |
|---|---|
| Bad deploy (health check fails) | Automatic: `rollback.sh` to last good SHA |
| Bad change discovered later | `git revert` + merge — which *is* a deploy |
| Bad platform state, cause unclear | Redeploy last release tag: `deploy.sh <repo> <stack>` at `vX.Y.Z` |
