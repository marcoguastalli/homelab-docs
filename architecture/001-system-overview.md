# 001 — System Overview

## One-diagram summary

```text
                          INTERNET
                              │
                              │  UDP 51820 only (router port-forward)
                    ┌─────────▼──────────┐
                    │      Router        │  DHCP hands out DNS = Pi-hole
                    └─────────┬──────────┘
                              │ LAN 192.168.1.0/24
              ┌───────────────┼──────────────────────┐
              │               │                      │
        LAN clients     VPN clients (10.8.0.0/24)    │
              │               │                      │
              └───────┬───────┘                      │
                      │  *.kirito.com → Pi LAN IP (Pi-hole split-horizon)
        ┌─────────────▼─────────────────────────────────────────────┐
        │  RASPBERRY PI 5 (8 GB, M.2 SSD)                           │
        │                                                           │
        │  Host (bash-provisioned): Docker, UFW, SSH, GH runner     │
        │                                                           │
        │  ┌─ INFRASTRUCTURE LAYER ──────────────────────────────┐  │
        │  │ Traefik :443/:80   Authelia   Pi-hole :53   wg-easy │  │
        │  │            shared network: net_proxy                │  │
        │  └──────────────────────────┬──────────────────────────┘  │
        │                             │  (Traefik ↔ every service)  │
        │  ┌─ SERVICES LAYER ─────────▼──────────────────────────┐  │
        │  │ Homepage  Dockge(RO)  Monitoring stack              │  │
        │  │ Uptime Kuma  Duplicati  …future services            │  │
        │  └─────────────────────────────────────────────────────┘  │
        │                                                           │
        │  /srv/homelab/data     ← all persistent state (SSD)       │
        │  /srv/homelab/secrets  ← .env files (never in Git)        │
        │  /mnt/backup           ← USB pendrive (Duplicati target)  │
        └───────────────────────────────────────────────────────────┘
                              ▲
                              │ deploys (self-hosted runner, push on main)
                    ┌─────────┴──────────┐
                    │      GitHub        │  5 repos · CI on hosted runners
                    └────────────────────┘
```

## The single most important invariant

**No container publishes a host port except Traefik (80/443), Pi-hole (53)
and WireGuard (51820/udp).** Every other service is reachable only through
Traefik over the internal `net_proxy` Docker network. A service that
publishes its own port bypasses both Traefik *and* the firewall
(see [ADR-004](../adr/ADR-004-docker-networking.md)) and will not pass
review.

## Core loop

1. Edit a compose file / config on a branch.
2. GitHub-hosted runners validate (lint, `compose config`, secret scan).
3. Merge to `main` (protected branch, CI required).
4. The self-hosted runner on the Pi deploys **only the changed stacks**
   via `homelab-ops/scripts/deploy/deploy.sh`.
5. Health checks gate the deploy; failure triggers automatic rollback.
6. Prometheus, Uptime Kuma and nightly CI watch it from then on.

## Layers and their repositories

| Layer | Deployed by | Contents |
|---|---|---|
| Host | [homelab-bootstrap](https://github.com/marcoguastalli/homelab-bootstrap) (manual, rare) | OS config, Docker, UFW, mounts, users, Actions runner |
| Infrastructure | [homelab-infrastructure](https://github.com/marcoguastalli/homelab-infrastructure) (CI) | Traefik, Authelia, Pi-hole, WireGuard, shared networks, TLS |
| Services | [homelab-services](https://github.com/marcoguastalli/homelab-services) (CI) | One directory per workload stack |
| Operations | [homelab-ops](https://github.com/marcoguastalli/homelab-ops) (consumed by CI) | Deploy/health/backup scripts, reusable workflows, scheduled jobs |
| Documentation | homelab-docs (this repo) | Everything you are reading |

Deployment order is strict: bootstrap → infrastructure → services.
`deploy.sh` refuses to deploy a service when the `net_proxy` network does
not exist.

## Current service catalog

See [005-platform-services.md](005-platform-services.md) for the
authoritative catalog, subdomains and RAM budget.
