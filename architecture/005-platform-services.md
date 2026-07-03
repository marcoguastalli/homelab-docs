# 005 — Platform Services

Authoritative catalog of everything running on the Pi, with subdomains,
auth policy and RAM budget. **A PR that adds or removes a service MUST
update this document.**

## Service catalog

| Subdomain | Service | Repo / stack | Auth policy | Purpose |
|---|---|---|---|---|
| `auth.kirito.com` | Authelia | infrastructure/authelia | bypass (login page) | SSO + 2FA portal |
| `traefik.kirito.com` | Traefik dashboard | infrastructure/traefik | two_factor | Ingress introspection |
| `dns.kirito.com` | Pi-hole admin | infrastructure/pihole | two_factor | DNS + ad-blocking admin |
| `vpn.kirito.com` | wg-easy UI | infrastructure/wireguard | two_factor | WireGuard peer management |
| `home.kirito.com` | Homepage | services/homepage | one_factor | Dashboard / launcher |
| `dockge.kirito.com` | Dockge (read-only) | services/dockge | two_factor | Stack/log observability ([ADR-015](../adr/ADR-015-dockge-read-only.md)) |
| `grafana.kirito.com` | Grafana | services/monitoring | one_factor | Dashboards |
| `prometheus.kirito.com` | Prometheus | services/monitoring | two_factor | Metrics DB |
| `alerts.kirito.com` | Alertmanager | services/monitoring | two_factor | Alert routing |
| `status.kirito.com` | Uptime Kuma | services/uptime-kuma | one_factor | Black-box checks + status page |
| `backup.kirito.com` | Duplicati | services/duplicati | two_factor | Backup management |

Non-HTTP listeners: Pi-hole `:53`, WireGuard `:51820/udp` — the only
services with published host ports besides Traefik.

## Naming rule

One name everywhere: stack directory = compose project = container name =
Traefik router = subdomain
([STD-001](../standards/STD-001-naming-conventions.md)).

## RAM budget (8 GB total)

| Layer | Stacks | Budget |
|---|---|---|
| Host + Docker + runner | — | ~800 MB |
| Infrastructure | Traefik, Authelia, Pi-hole, wg-easy | ~450 MB |
| Observability | monitoring stack, Uptime Kuma | ~1.3 GB |
| Platform services | Homepage, Dockge, Duplicati | ~500 MB |
| **Free for future services** | — | **~5 GB** |

Rules:

- Every container MUST declare `mem_limit`
  ([STD-002](../standards/STD-002-compose-conventions.md)).
- The budget table above MUST be updated when a service is added.
- **Heavy candidates** (Immich ML, Ollama, Jellyfin transcoding, Nextcloud)
  can each consume the entire remaining budget — adopting one REQUIRES an
  ADR that states its measured footprint and what it displaces.

## Future service intake checklist

1. Copy `homelab-services/_template/` → new stack directory.
2. Pin the image tag; set `mem_limit`, healthcheck, labels.
3. Add the subdomain row to this document + Authelia access-control rule.
4. Create `/srv/homelab/secrets/<stack>.env` on the Pi from `.env.example`.
5. If stateful: define its consistent-dump command
   ([007](007-backup-disaster-recovery.md)) and set `kirito.backup: "true"`.
6. Merge → deploy → verify → add Uptime Kuma check.

Full procedure: [RB-002](../runbooks/RB-002-add-a-new-service.md).
