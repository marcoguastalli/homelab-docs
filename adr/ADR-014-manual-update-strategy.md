# ADR-014 — Manual Update Strategy

- Status: Accepted
- Date: 2026-07-03

## Context

Pinned image versions ([STD-002](../standards/STD-002-compose-conventions.md))
need a process for moving forward. Options ranged from fully automatic
(Watchtower) through bot-assisted PRs (Renovate) to manual. The user
chose manual, with automation limited to *reporting*.

## Decision

- All image references are **pinned to exact versions**; `latest` is
  banned.
- Updates are applied **manually** during a monthly maintenance window
  ([RB-006](../runbooks/RB-006-update-services.md),
  [architecture/011](../architecture/011-operations-model.md)): one PR per
  stack bumping the pin → CI → merge → auto-deploy → verify.
- Automation keeps the process honest without taking the decision:
  - `update-report.yml` (weekly): compares every pinned tag against its
    registry (skopeo) and maintains a single "available updates" GitHub
    issue.
  - `nightly-validation.yml`: Trivy-scans all pinned images; HIGH/CRITICAL
    findings on ingress/auth/VPN components are patched **immediately**,
    not at the next window.
- **OS security patches are the exception**: unattended-upgrades
  (security pocket only) — CVE latency at the OS layer is unacceptable.

## Consequences

- Every change to running software is a reviewed, revertible commit;
  the deployed state never surprises the operator.
- Cost: sustained discipline. The weekly report issue and the calendar
  window are the guardrails; if update PRs pile up unmerged, that is the
  signal to adopt Renovate.
- Update latency of up to a month for non-security releases — accepted.

## Alternatives considered

- **Watchtower** — rejected: updates outside Git break the source-of-truth
  model, no pre-deploy validation, no audit trail, 3 a.m. breakage.
- **Renovate PRs** — the architecture's recommendation (same control,
  less toil); declined by user decision after explanation. Listed as
  improvement #4; trivially adoptable later since pins are already the
  format Renovate edits.
