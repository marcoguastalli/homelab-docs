# 000 — Architecture Vision

## Purpose

A production-grade, self-hosted platform on a single Raspberry Pi 5 that a
single maintainer can operate confidently for ten years — and that a
*different* maintainer could pick up from these documents alone.

## Goals

1. **Reproducibility** — the entire platform (minus runtime data) can be
   rebuilt from Git plus one encrypted backup. Disaster recovery is a
   documented, drilled, ~2-hour procedure, not an archaeology project.
2. **Modularity** — adding a service touches one directory; breaking a
   service breaks only that service.
3. **Security by default** — nothing is exposed to the internet except a
   single WireGuard UDP port; every admin surface sits behind SSO + 2FA;
   a new service is private until it explicitly opts in to ingress.
4. **Documentation as code** — prose lives in Git, is linted by CI, and
   records *why*, not just *what*. Decisions are append-only ADRs.
5. **Maintainability over cleverness** — boring, explicit, convention-driven
   choices win over minimal-file-count or novel tooling.

## Non-goals

- **High availability.** One Pi is a single point of failure and that is
  accepted. The mitigation is fast, reliable recovery — not redundancy.
- **Public hosting.** No service is reachable from the internet. Serving
  friends/family outside the VPN is out of scope until an ADR changes it.
- **Kubernetes.** Docker Compose is the deliberate ceiling for one node
  (see [ADR-001](../adr/ADR-001-repository-boundaries.md) scope notes).
- **Zero-touch automation.** Updates are manual by decision
  ([ADR-014](../adr/ADR-014-manual-update-strategy.md)); the platform
  automates validation, deployment, monitoring and reporting — a human
  makes the change decisions.

## Principles

| Principle | Concrete meaning here |
|---|---|
| Infrastructure as Code | Everything except runtime data and secrets is in Git |
| GitOps | `main` is what is deployed; deploys are triggered by merges |
| Separation of concerns | Five repos split by rate of change and blast radius |
| Single responsibility | One stack = one directory = one deployable unit |
| Least privilege | No `privileged` containers, read-only socket mounts, UFW default-deny |
| Convention over configuration | `_template/` service skeleton; one name for stack/container/subdomain |
| Declarative & idempotent | Compose is declarative; bootstrap stages and deploy scripts are re-runnable |
| Immutable where applicable | Pinned image tags; ADRs and changelogs append-only |
| Documentation as code | This repo, linted and link-checked by CI |

## Constraints

| Constraint | Value |
|---|---|
| Hardware | Raspberry Pi 5, 8 GB RAM, M.2 SSD (NVMe), arm64 |
| OS | Raspberry Pi OS (Debian-based) |
| Exposure | LAN + WireGuard VPN only |
| RAM budget | See [005-platform-services.md](005-platform-services.md) — RAM, not architecture, is the scaling limit |
| Maintainer | One person; processes sized accordingly ([ADR-009](../adr/ADR-009-versioning.md), [ADR-010](../adr/ADR-010-release-strategy.md)) |

## Accepted risks (deliberate, documented)

These were chosen consciously and each has an ADR and an upgrade path:

1. **No off-site backup** — single encrypted USB target
   ([ADR-007](../adr/ADR-007-backup-strategy.md)). Highest-priority listed
   improvement.
2. **Self-signed TLS** — one-time trust ritual per device
   ([ADR-011](../adr/ADR-011-self-signed-tls.md)).
3. **Plain `.env` secrets on the Pi only** — disaster recovery depends
   entirely on the backup plus passphrase escrow
   ([ADR-012](../adr/ADR-012-plain-env-secrets.md)).
