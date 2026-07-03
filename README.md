# homelab-docs

Documentation for the **kirito.com homelab platform** — a Git-driven,
Docker-Compose-based self-hosted platform running on a Raspberry Pi 5 (8 GB),
reachable only via LAN and WireGuard VPN.

This repository is the **single source of truth for prose**: architecture,
decisions, standards and operational procedures. It contains nothing
executable and nothing deployable.

## The five repositories

| Repository | Owns | Rate of change |
|---|---|---|
| [homelab-bootstrap](https://github.com/marcoguastalli/homelab-bootstrap) | The host: OS, Docker, firewall, mounts, users, CI runner | Rare |
| [homelab-infrastructure](https://github.com/marcoguastalli/homelab-infrastructure) | The platform plane: Traefik, Authelia, Pi-hole, WireGuard, networks, TLS | Occasional |
| [homelab-services](https://github.com/marcoguastalli/homelab-services) | Every workload stack, one directory per service | Frequent |
| [homelab-ops](https://github.com/marcoguastalli/homelab-ops) | Executable operations: deploy, health, backup-verify, reusable CI | Occasional |
| **homelab-docs** (this repo) | Architecture, ADRs, runbooks, standards | Continuous |

## Reading order for a new maintainer

1. [architecture/000-architecture-vision.md](architecture/000-architecture-vision.md) — goals, non-goals, principles
2. [architecture/001-system-overview.md](architecture/001-system-overview.md) — the big picture in one diagram
3. [architecture/002-repository-model.md](architecture/002-repository-model.md) — why five repos, and their contracts
4. The remaining numbered documents in [architecture/](architecture/)
5. [adr/](adr/README.md) — the decision record, including the consciously accepted trade-offs
6. [standards/](standards/) — before writing any code or compose file
7. [runbooks/](runbooks/) — when operating the platform

## Directory map

```
architecture/   Numbered architecture documents (the "what and how")
adr/            Architecture Decision Records (the "why") — append-only
decisions/      Draft and rejected proposals (pre-ADR stage)
diagrams/       Diagram sources (.drawio/.excalidraw) + exported .svg
runbooks/       Step-by-step operational procedures (RB-xxx)
standards/      Conventions and coding standards (STD-xxx)
```

## Documentation rules

- Every change lands via pull request; CI runs markdownlint and a link check.
- ADRs are immutable once accepted: supersede, never edit.
- Runbooks are tested by use — if a runbook step is wrong, fixing it is part
  of the incident.
- Standards documents are normative: MUST / SHOULD / MAY per
  [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

See [STD-004 Documentation Standards](standards/STD-004-documentation-standards.md)
for authoring rules, and [CONTRIBUTING.md](CONTRIBUTING.md) for workflow.

## License

[MIT](LICENSE)
