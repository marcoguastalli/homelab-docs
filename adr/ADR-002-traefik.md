# ADR-002 — Traefik as Sole Ingress

- Status: Accepted
- Date: 2026-07-03

## Context

Dozens of HTTP services on one host need TLS, consistent hostnames,
central auth enforcement and zero per-service port management. The
reverse proxy is the platform's most load-bearing component.

## Decision

**Traefik v3** is the single reverse proxy and the only container
publishing 80/443.

- Docker provider with `exposedByDefault: false` — services opt in via
  labels; a new service is invisible until it explicitly enables ingress.
- File provider for dynamic config that isn't service-specific: TLS
  defaults, shared middlewares (`authelia@file`, `secured@file`).
- Configuration conventions are normative in
  [STD-003](../standards/STD-003-traefik-conventions.md).

## Consequences

- Adding ingress for a service = 4 labels in its own compose file; auth
  policy changes happen in one middleware file, never per service.
- Traefik needs the Docker socket (read-only) — accepted, mitigated in
  [architecture/004](../architecture/004-security-architecture.md);
  `docker-socket-proxy` is a listed future improvement.
- Traefik down = total HTTP outage. Accepted: single node means a single
  ingress SPOF regardless of proxy choice; mitigation is monitoring + fast
  rollback.

## Alternatives considered

- **nginx / nginx-proxy-manager** — rejected: hand-written or UI-managed
  per-service config duplicates state outside Git (NPM's UI is the same
  anti-pattern as Dockge-as-authority, [ADR-015](ADR-015-dockge-read-only.md)).
- **Caddy** — strong contender (simpler config, automatic HTTPS), rejected
  on weaker Docker-label auto-discovery and a smaller middleware ecosystem
  (forwardAuth patterns with Authelia are first-class in Traefik).
