# ADR-005 — Monitoring Stack

- Status: Accepted
- Date: 2026-07-03

## Context

A platform meant to run for years needs to answer both "is it broken?"
(user view) and "why is it broken?" (operator view), plus alert without
depending on the thing being monitored. Budget: ~1.3 GB RAM on an 8 GB
host.

## Decision

- **Prometheus + node-exporter + cAdvisor + Alertmanager** — white-box
  metrics and alerting; 15-day retention, 30 s scrape.
- **Grafana** — dashboards, provisioned entirely from JSON in Git.
- **Uptime Kuma** — independent black-box HTTP checks on every endpoint,
  plus the `status.kirito.com` page.
- **ntfy** as the notification channel (push over VPN, no SMTP
  dependency). Critical → immediate push; warning → daily digest.
- All alert rules version-controlled
  (`homelab-services/monitoring/prometheus/rules/`).
- Deployed as **one compose stack** (single failure domain, configs
  version together) — not five directories that must change in lockstep.

## Consequences

- ~1.3 GB RAM for observability — accepted as the price of operating
  confidently; it is the second-largest consumer after free headroom.
- White-box and black-box monitors fail independently: Prometheus dying
  doesn't blind the status page, and vice versa.
- No log aggregation (Loki) and no tracing initially — single node,
  `docker logs` + Dockge read-only suffice; revisit by proposal.

## Alternatives considered

- **Netdata** — beautiful out of the box, rejected: heavier per-node
  agent, weaker alert-rule-as-code story.
- **Uptime Kuma only** — rejected: tells you *that* it broke, never *why*;
  no host/container metrics for capacity decisions (RAM budget needs data).
- **VictoriaMetrics** — lighter than Prometheus, rejected for now: smaller
  ecosystem/dashboards; a drop-in future swap if Prometheus RAM becomes a
  problem.
- **Email alerts** — rejected: requires SMTP credentials and delivery
  debugging; ntfy is simpler and VPN-friendly.
