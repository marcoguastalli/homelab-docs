# Glossary

| Term | Meaning in this project |
|---|---|
| ADR | Architecture Decision Record — immutable, numbered record of an accepted decision, in [adr/](../adr/) |
| Blast radius | What breaks if this component/repo is broken — the repo-boundary criterion |
| Contract | An explicitly documented coupling between two repos ([002](002-repository-model.md)) |
| `DATA_ROOT` | Env var (`/srv/homelab/data`) under which **all** persistent bind mounts live |
| DR kit | The off-Pi set of items needed for disaster recovery ([007](007-backup-disaster-recovery.md)) |
| forwardAuth | Traefik middleware delegating each request's auth decision to Authelia |
| GitOps | `main` is what is deployed; merges trigger deploys; Git is the only write path |
| Infrastructure layer | Traefik + Authelia + Pi-hole + WireGuard + shared networks — what all services depend on |
| Maintenance window | The monthly manual-update session ([011](011-operations-model.md)) |
| `net_proxy` | The shared Docker network connecting Traefik to every HTTP service |
| Pin / pinned tag | An exact image version (`grafana:11.3.1`) — `latest` is banned |
| Runbook (RB-xxx) | Step-by-step operational procedure in [runbooks/](../runbooks/) |
| Self-hosted runner | GitHub Actions runner on the Pi; deploys only, `main` only |
| Split-horizon DNS | `*.kirito.com` resolves to the Pi's LAN IP on LAN/VPN and nowhere else |
| Stack | One compose project = one directory = one deployable unit |
| Standard (STD-xxx) | Normative convention document in [standards/](../standards/) (RFC-2119 language) |
| Two-factor / one-factor | Authelia access policies — 2FA for admin surfaces, password for user apps |
