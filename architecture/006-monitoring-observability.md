# 006 — Monitoring & Observability

## Topology

```text
node-exporter (host) ──┐
cAdvisor (containers) ─┼──▶ Prometheus ──▶ Alertmanager ──▶ ntfy
Traefik /metrics ──────┤        │
Authelia /metrics ─────┘        ▼
                             Grafana (dashboards provisioned from Git)

Uptime Kuma ── HTTP checks on every *.kirito.com endpoint (black-box, user view)
           └── status.kirito.com status page
```

White-box (Prometheus: *why* is it broken) and black-box (Uptime Kuma:
*is* it broken, as a user sees it) are deliberately both present —
they fail independently. Stack choice rationale:
[ADR-005](../adr/ADR-005-monitoring-stack.md).

## Coverage matrix

| Concern | Tool | Key alerts (version-controlled in `monitoring/prometheus/rules/`) |
|---|---|---|
| Host | node-exporter | disk >80 %, memory >90 % sustained 10 min, CPU temp >75 °C, SSD SMART failure |
| Containers | cAdvisor | OOM-killed, restart loop (>3 in 15 min), expected stack absent |
| Ingress | Traefik metrics | 5xx rate, request latency, cert expiry |
| Auth | Authelia metrics | authentication failure spike |
| Endpoints | Uptime Kuma | any endpoint down >2 min |
| Backups | Duplicati status → textfile exporter | **no successful backup in >26 h** — the alert that matters most |
| CI | homelab-ops `nightly-validation.yml` | `main` of any repo fails validation |

## Notification channel

**ntfy** (push notifications; works over VPN, no SMTP dependency).
Alertmanager routes:

- `severity: critical` → immediate push (backup failure, disk, service down)
- `severity: warning` → daily digest
- `severity: info` → no push; visible in Grafana/Alertmanager only

## Retention & footprint

- Prometheus retention: **15 days** (SSD-friendly; long-term trends are not
  a goal). Scrape interval 30 s.
- Total observability budget ~1.3 GB RAM
  ([005](005-platform-services.md)) — accepted as the price of operating
  confidently.

## Dashboards as code

Grafana dashboards and datasources are provisioned from JSON files in
`homelab-services/monitoring/grafana/provisioning/`. Dashboards edited in
the UI MUST be exported and committed to be kept — a rebuilt Pi produces
identical dashboards with zero clicking.

## What is deliberately absent

- **Log aggregation (Loki):** not in the initial platform. `docker logs`
  plus Dockge's read-only log view cover the single-node case. Revisit via
  proposal when debugging across services becomes painful.
- **Tracing:** out of scope for household workloads.
