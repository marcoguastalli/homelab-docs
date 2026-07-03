# STD-001 — Naming Conventions

Normative. Key words MUST/SHOULD/MAY per RFC 2119.

## The one-name rule

A service MUST use the same lowercase name everywhere:

```text
stack directory = compose project (name:) = container name
= Traefik router = Traefik service = subdomain = secrets file
```

Example — `grafana`: directory `homelab-services/monitoring/` (multi-
container stacks use the *stack* name), container `grafana`, router
`grafana`, `grafana.kirito.com`, part of `monitoring.env`.

## Rules by object

| Object | Convention | Example |
|---|---|---|
| Repository | `homelab-<purpose>` | `homelab-services` |
| Stack directory | lowercase, hyphenated, singular purpose | `uptime-kuma` |
| Container | stack name; multi-container: `<stack>-<component>` | `monitoring-prometheus` → but single-purpose containers keep their own name (`prometheus`) when unambiguous |
| Docker network (shared) | `net_<purpose>` | `net_proxy`, `net_monitoring` |
| Docker network (private) | `<stack>_internal` | `immich_internal` |
| Data directory | `${DATA_ROOT}/<stack>/<component>` | `${DATA_ROOT}/monitoring/grafana` |
| Secrets file | `/srv/homelab/secrets/<stack>.env` | `/srv/homelab/secrets/monitoring.env` |
| Subdomain | `<service>.kirito.com`, short, functional | `dns.` not `pihole.`; `vpn.` not `wg-easy.` |
| Env variable | `UPPER_SNAKE`, prefixed by component when shared | `GRAFANA_ADMIN_PASSWORD` |
| Branch | `feat/`, `fix/`, `docs/`, `chore/` + kebab topic | `feat/add-vaultwarden` |
| Tag | `vMAJOR.MINOR.PATCH` | `v1.4.0` |
| Custom labels | `kirito.<key>` namespace | `kirito.backup: "true"` |
| Runbooks / standards / ADRs | `RB-NNN-`, `STD-NNN-`, `ADR-NNN-` + kebab slug | `RB-004-full-disaster-recovery.md` |
| Scripts | verb-first kebab-case `.sh` | `health-check.sh` |

## Subdomain registry

The authoritative subdomain table is
[architecture/005](../architecture/005-platform-services.md). A PR adding
a subdomain MUST update it. Subdomains SHOULD name the *function*
(`photos.`, `vault.`, `status.`), not the product — products get
replaced; functions don't.
