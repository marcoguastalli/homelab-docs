# ADR-004 — Docker Networking Model

- Status: Accepted
- Date: 2026-07-03

## Context

~30 future services must be reachable through Traefik, scrapeable by
Prometheus, and isolated from each other's internals — without a network
topology that becomes unmaintainable. Additionally, Docker's published
ports bypass UFW by writing iptables rules directly: the most common
homelab security hole.

## Decision

1. **One shared `net_proxy` network** (external, created by
   infrastructure) joining Traefik and every HTTP service.
2. **One shared `net_monitoring` network** for Prometheus and its scrape
   targets.
3. **Per-stack `<stack>_internal` networks** for anything not meant for
   ingress (databases, queues).
4. **No published host ports** except Traefik (80/443), Pi-hole (53),
   WireGuard (51820/udp) — the platform's core invariant.
5. Bootstrap installs firewall rules in the **`DOCKER-USER`** iptables
   chain (which Docker respects) mirroring the UFW policy, so even a
   mistakenly published port is not internet/LAN-open by accident.

## Consequences

- East-west: a service on `net_proxy` can technically reach other
  `net_proxy` members' exposed HTTP ports. Accepted because nothing
  sensitive listens there — DBs live on `_internal` networks; admin
  surfaces still require Authelia.
- Adding a service = join `net_proxy` + labels; zero topology changes.
- The DOCKER-USER rules must be maintained in bootstrap alongside UFW
  rules; stage `70-verify.sh` asserts both.

## Alternatives considered

- **Per-service proxy networks (Traefik joins ~30 networks)** — maximum
  isolation, rejected: unmaintainable network sprawl, Traefik container
  needs reconfiguring for every service, and the marginal gain over
  "_internal for sensitive components" is small at this threat model.
- **Host networking** — rejected outright: no isolation, port conflicts,
  bypasses the entire model.
- **Single flat network for everything** — rejected: databases reachable
  from every container.
