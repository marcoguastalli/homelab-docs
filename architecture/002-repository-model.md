# 002 — Repository Model

## Boundary rule

Repositories are split by **rate of change** and **blast radius**, not by
technology. Full rationale in
[ADR-001](../adr/ADR-001-repository-boundaries.md).

| Repository | Owns | Rate of change | Blast radius if broken | Never contains |
|---|---|---|---|---|
| homelab-bootstrap | The host itself: OS, Docker, firewall, mounts, users, runner | Rare (months) | Everything | Compose files, service config |
| homelab-infrastructure | Platform plane: Traefik, Authelia, Pi-hole, WireGuard, shared networks, TLS generation | Occasional (weeks) | All services unreachable | Application services, host provisioning |
| homelab-services | Every workload stack, one directory per service | Frequent (daily/weekly) | One service | Ingress/auth/DNS config, host scripts |
| homelab-ops | Executable operations: deploy/rollback/health/backup-verify/update-report, reusable CI workflows | Occasional | Automation only (platform keeps running) | Service definitions, prose documentation |
| homelab-docs | Prose: architecture, ADRs, runbooks, standards | Continuous | Nothing (humans only) | Anything executable or deployable |

## Dependency graph

```text
                 ┌──────────────────┐
                 │  homelab-docs    │   referenced by all (prose links),
                 │  (no code deps)  │   depends on none
                 └──────────────────┘

  provisions                 deploys onto                deploys onto
┌───────────────┐  host   ┌──────────────────┐ networks+ ┌──────────────┐
│ homelab-      │─ready──▶│ homelab-         │─Traefik──▶│ homelab-     │
│ bootstrap     │         │ infrastructure   │ +Authelia │ services     │
└───────┬───────┘         └────────┬─────────┘           └──────┬───────┘
        │ installs runner          │ CI/deploy uses             │ CI/deploy uses
        ▼                          ▼                            ▼
                    ┌───────────────────────────────┐
                    │         homelab-ops           │
                    │ reusable workflows + scripts  │
                    └───────────────────────────────┘
```

## Contracts (the only allowed couplings)

Anything crossing a repository boundary that is not in this table is an
architecture violation and needs an ADR.

| Producer | Contract | Consumer |
|---|---|---|
| bootstrap | Filesystem layout `/srv/homelab/*`; Docker installed and configured; runner registered; `/mnt/backup` mounted | infrastructure, services, ops |
| infrastructure | External Docker networks `net_proxy`, `net_monitoring`; Traefik middlewares `authelia@file`, `secured@file`; wildcard TLS cert in `/srv/homelab/secrets/tls/` | services |
| ops | Script CLIs: `deploy.sh <repo> <stack>`, `rollback.sh <repo> <stack> <git-ref>`, `health-check.sh <stack>`; reusable workflows `reusable-validate.yml`, `reusable-deploy.yml` (`workflow_call`) | bootstrap, infrastructure, services |
| services | Prometheus scrape labels; Homepage discovery labels; `kirito.backup` label | infrastructure (monitoring/backup discover them) |

Breaking a contract requires: an ADR, a **major** version bump of the
producer repo, and a migration note in each consumer's changelog
(see [010-release-management.md](010-release-management.md)).

## Host filesystem layout (bootstrap's contract, reference copy)

```text
/srv/homelab/
├── repos/      # git clones used by the runner (infrastructure, services, ops)
├── data/       # ALL persistent service state; DATA_ROOT env var points here
├── secrets/    # per-stack .env files + tls/ — mode 0700, never in Git
└── state/      # deploy bookkeeping (<stack>.deployed = last good SHA)

/mnt/backup     # USB pendrive — Duplicati target
/mnt/data       # reserved: future external USB SSD (see 007-backup-disaster-recovery.md)
```

## Why not a monorepo?

A monorepo was rejected because the layer touched daily (services) would
share history, CI triggers and merge risk with the layer that can take the
whole platform down (infrastructure) and the layer that can break the host
(bootstrap). Path-filtered CI can simulate the separation, but branch
protection, release versioning and blast-radius isolation cannot be
simulated per-directory. Trade-off and alternatives in
[ADR-001](../adr/ADR-001-repository-boundaries.md).
