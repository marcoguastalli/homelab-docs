# ADR-015 — Dockge in Read-Only Role

- Status: Accepted
- Date: 2026-07-03

## Context

Dockge is a compose-stack manager whose main feature is *editing and
deploying stacks through a web UI*. That capability directly contradicts
the platform's core rule: **Git is the only write path** — every change
to running services must be a commit that CI validated and the deploy
pipeline applied.

## Decision

Dockge is deployed **strictly as an observability tool**:

- Stack directories are mounted **read-only** into the Dockge container.
- Its role: at-a-glance stack status, container logs, quick visual
  debugging — a convenience layer between `docker ps` and Grafana.
- Its UI sits behind Authelia `two_factor`
  ([architecture/005](../architecture/005-platform-services.md)).
- Any stack edit or lifecycle action goes through a PR, never through
  Dockge. If a Dockge version/feature circumvents the read-only mount,
  it is removed from the platform.

## Consequences

- Keeps the ergonomic benefit (fast log/status view on any device) at
  zero drift risk.
- Dockge still sees the Docker socket (it must, to list stacks) — the
  socket exposure is accepted for observability, consistent with cAdvisor,
  and listed under socket-hardening improvements
  ([architecture/004](../architecture/004-security-architecture.md)).

## Alternatives considered

- **Dockge as deployment authority, Git mirrors it** — rejected outright:
  inverts the source of truth; "the UI and Git disagree" becomes a
  permanent failure mode with no CI gate.
- **Drop Dockge entirely** (Homepage + Grafana suffice) — viable and
  simpler; kept because read-only log access from a phone during an
  incident is genuinely useful. Re-evaluate if the read-only constraint
  ever becomes hard to maintain.
- **Portainer** — same authority conflict, larger surface; rejected for
  the same reason plus weight.
