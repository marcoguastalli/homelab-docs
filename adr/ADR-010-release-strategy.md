# ADR-010 — Release Strategy

- Status: Accepted
- Date: 2026-07-03

## Context

Deployment is continuous (every merge deploys), so "release" must mean
something else useful — or nothing. Disaster recovery and debugging need
the ability to name and restore a known-good platform state.

## Decision

- **Releases are tags marking known-good states**, not deployment events.
- Tag `vX.Y.Z` per repo after meaningful milestones and after each
  monthly maintenance window, once health checks and monitoring confirm
  stability.
- `release.yml` automates the rest: git-cliff changelog from conventional
  commits → GitHub Release.
- Breaking cross-repo changes follow the four-step procedure in
  [architecture/010](../architecture/010-release-management.md)
  (ADR → major bump → consumer migration notes → same-window update).

## Consequences

- "Restore platform as of `infrastructure v1.4.0 + services v2.1.3`" is a
  precise, executable statement — `deploy.sh` accepts any git ref.
- Rollback layers: automatic (failed health check) → `git revert` →
  redeploy last release tag, in increasing severity
  ([architecture/010](../architecture/010-release-management.md)).
- Cost: remembering to tag. Mitigated by making it the closing step of
  the monthly maintenance runbook
  ([RB-006](../runbooks/RB-006-update-services.md)).

## Alternatives considered

- **Release = every deploy** — rejected: hundreds of meaningless tags.
- **Calendar releases (monthly)** — partially adopted: the maintenance
  window naturally produces a tag, but milestones outside the window also
  deserve one; strict CalVer adds no information SemVer lacks here.
