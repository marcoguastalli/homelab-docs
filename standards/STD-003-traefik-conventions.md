# STD-003 — Traefik Conventions

Normative. Key words MUST/SHOULD/MAY per RFC 2119. Architecture context:
[ADR-002](../adr/ADR-002-traefik.md).

## Entrypoints

| Entrypoint | Port | Role |
|---|---|---|
| `web` | 80 | Only job: permanent redirect → `websecure`. Nothing routes on `web`. |
| `websecure` | 443 | All traffic; TLS with the default self-signed wildcard ([ADR-011](../adr/ADR-011-self-signed-tls.md)) |

## Configuration split

| Kind | Where | Examples |
|---|---|---|
| Static config | `traefik/config/traefik.yaml` | entrypoints, providers, metrics, `exposedByDefault: false` |
| Shared dynamic config | `traefik/config/dynamic/*.yaml` (file provider, watched) | default TLS cert, `authelia@file`, `secured@file` |
| Per-service routing | Labels in the service's own compose file | router rule, entrypoint, middlewares, port |

Per-service routing MUST live in the service's labels — never in the file
provider. Shared policy MUST live in the file provider — never duplicated
in labels.

## Required labels per HTTP service

```yaml
traefik.enable: "true"                                        # opt-in (exposedByDefault: false)
traefik.http.routers.<name>.rule: Host(`<name>.kirito.com`)
traefik.http.routers.<name>.entrypoints: websecure
traefik.http.routers.<name>.middlewares: authelia@file        # per auth policy, see below
traefik.http.services.<name>.loadbalancer.server.port: "<internal-port>"
```

`<name>` MUST equal the stack/service name (STD-001).
`loadbalancer.server.port` MUST always be explicit — auto-detection
breaks silently on multi-port images.

## Middlewares

| Middleware | Contents | Applied to |
|---|---|---|
| `secured@file` | Security headers (X-Frame-Options, nosniff, referrer policy; **no HSTS** while self-signed), rate limiting | Every router (Traefik `defaultMiddlewares` where possible) |
| `authelia@file` | forwardAuth → Authelia `/api/verify` | Per the auth-policy table in [architecture/005](../architecture/005-platform-services.md): `two_factor` admin surfaces MUST carry it; `one_factor` apps per policy |

New shared middlewares MUST be added to the file provider and this
document in the same PR.

## Dashboard

`traefik.kirito.com`, `api.insecure: false`, behind `authelia@file`
(two_factor). The dashboard MUST never be exposed via the insecure API
port.
