# ADR-001 — Repository Boundaries

- Status: Accepted
- Date: 2026-07-03

## Context

The platform needs version control for host provisioning, platform
services (ingress/auth/DNS/VPN), workload services, operational
automation and documentation. These have very different rates of change
and very different blast radii when broken. A careless merge in code
touched daily must not be able to take down the ingress layer or
re-provision the host.

## Decision

Five repositories, split by **rate of change** and **blast radius**:

1. `homelab-bootstrap` — the host (pre-Docker execution context)
2. `homelab-infrastructure` — platform plane every service depends on
3. `homelab-services` — workloads; one directory per stack
4. `homelab-ops` — shared executable operations + reusable CI workflows
5. `homelab-docs` — prose only

Couplings between repos are limited to explicitly documented contracts
([architecture/002](../architecture/002-repository-model.md)).

## Consequences

- Independent branch protection, versioning and release cadence per layer.
- A services PR physically cannot modify Traefik or the firewall.
- Cost: cross-repo changes (rare, contract-breaking ones) need coordinated
  PRs and are governed by the breaking-change procedure in
  [architecture/010](../architecture/010-release-management.md).
- `homelab-ops` exists so deploy/validate logic is written once; without
  it, infrastructure and services would duplicate CI and scripts, or one
  would falsely depend on the other.

## Alternatives considered

- **Monorepo with path-filtered CI** — rejected: simulates separate
  pipelines but cannot simulate per-layer branch protection, versioning or
  blast-radius isolation; history of daily service tweaks buries rare
  host-level changes.
- **Two repos (infra + services)** — rejected: leaves ops scripts homeless
  (duplicated or misplaced) and mixes host provisioning with Docker-level
  platform config that runs in a different execution context.
- **Repo per service (~30 repos)** — rejected: unmaintainable overhead for
  a solo maintainer; services share conventions, CI and lifecycle.
