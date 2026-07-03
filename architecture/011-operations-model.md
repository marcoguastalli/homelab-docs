# 011 — Operations Model

How the platform is operated day to day, week to week. Update strategy
rationale: [ADR-014](../adr/ADR-014-manual-update-strategy.md).

## Cadences

| Cadence | Activity | Driven by |
|---|---|---|
| Continuous | Alerts (ntfy), auto-deploys on merge, auto-rollback | CI + monitoring |
| Daily 03:00 | Backup (dumps → Duplicati → pendrive) | cron on the Pi |
| Nightly 02:00 | Re-validate `main` of all repos; Trivy image scan | `nightly-validation.yml` |
| Weekly Sun 04:00 | Backup verification (canary restore) | `backup-verification.yml` |
| Weekly Mon 06:00 | Outdated-image report → GitHub issue | `update-report.yml` |
| Monthly | **Maintenance window** (below) | calendar + the Monday issue |
| Quarterly | DR tabletop drill; check DR kit exists | recurring issue |
| Annually | Real DR restore onto spare media | recurring issue |

## The monthly maintenance window

The core of the manual-update strategy
([RB-006](../runbooks/RB-006-update-services.md) is the executable
version):

1. Open the current *available updates* issue (maintained by
   `update-report.yml`).
2. For each update worth taking: read release notes — breaking changes?
3. One PR per stack (or a small batch): bump the image pin.
4. CI validates → merge → auto-deploy → health check → verify in Grafana /
   Uptime Kuma.
5. Anything broken: `git revert` (which redeploys the old pin).
6. Tag releases; close the issue; the next Monday report starts fresh.

**Security exception:** Trivy nightly findings at HIGH/CRITICAL on an
internet-facing-ish component (Traefik, Authelia, WireGuard) are handled
immediately, not at the next window. OS security patches are unattended
(the one auto-update exception — [004](004-security-architecture.md)).

## Operational interfaces

| Task | Interface |
|---|---|
| Change anything | Git PR (the only write path) |
| Observe stacks/logs | Dockge (read-only), `docker logs`, Grafana |
| Check platform health | `status.kirito.com` (Uptime Kuma), Grafana |
| Deploy manually (CI down) | `deploy.sh` locally on the Pi — designed to work without GitHub |
| Restore | Runbooks [RB-003](../runbooks/RB-003-restore-single-service.md) / [RB-004](../runbooks/RB-004-full-disaster-recovery.md) |

## Incident rule

If a runbook step turns out to be wrong during an incident, **fixing the
runbook is part of resolving the incident** — the PR that closes the
incident updates the runbook.
