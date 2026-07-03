# STD-002 — Docker Compose Conventions

Normative. Key words MUST/SHOULD/MAY per RFC 2119. The living example is
`homelab-services/_template/compose.yaml` — new stacks MUST start from it.

## Reference shape

```yaml
name: grafana                                   # explicit project name = dir name
services:
  grafana:
    image: grafana/grafana-oss:11.3.1           # pinned, never latest
    container_name: grafana
    restart: unless-stopped
    env_file: /srv/homelab/secrets/grafana.env
    security_opt: [no-new-privileges:true]
    mem_limit: 256m
    cpus: 1.0
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    volumes:
      - ${DATA_ROOT}/grafana:/var/lib/grafana
    networks: [net_proxy]
    labels:
      traefik.enable: "true"
      traefik.http.routers.grafana.rule: Host(`grafana.kirito.com`)
      traefik.http.routers.grafana.entrypoints: websecure
      traefik.http.routers.grafana.middlewares: authelia@file
      traefik.http.services.grafana.loadbalancer.server.port: "3000"
      homepage.group: Monitoring
      homepage.name: Grafana
      kirito.backup: "true"
networks:
  net_proxy: {external: true}
```

## Rules

| # | Rule | Why |
|---|---|---|
| 1 | File is `compose.yaml`; project `name:` MUST equal the directory name | One name everywhere (STD-001) |
| 2 | Image tags MUST be exact pins; `latest`/`stable` are banned | Reproducibility; manual-update model ([ADR-014](../adr/ADR-014-manual-update-strategy.md)) |
| 3 | `ports:` MUST NOT appear except in Traefik, Pi-hole, WireGuard | The core invariant; published ports bypass Traefik **and** UFW ([ADR-004](../adr/ADR-004-docker-networking.md)) |
| 4 | Persistent state MUST be a bind mount under `${DATA_ROOT}/<stack>/`; named volumes are banned | Backup scope + SSD migration see one tree ([architecture/007](../architecture/007-backup-disaster-recovery.md)) |
| 5 | `mem_limit` MUST be set; `cpus` SHOULD be set | 8 GB budget ([architecture/005](../architecture/005-platform-services.md)) |
| 6 | `healthcheck` MUST be defined | It is the deploy gate |
| 7 | `security_opt: [no-new-privileges:true]` MUST be set; `privileged` is banned; docker socket mounts read-only and only where unavoidable | Least privilege |
| 8 | Secrets via `env_file:` from `/srv/homelab/secrets/`; a complete `.env.example` MUST be committed | [ADR-008](../adr/ADR-008-secrets-management.md) |
| 9 | `restart: unless-stopped` | Survive reboots without fighting manual stops |
| 10 | HTTP services join `net_proxy` (external); internal components use `<stack>_internal`; both declared explicitly | [ADR-004](../adr/ADR-004-docker-networking.md) |
| 11 | Stateful services MUST define a consistent-dump command (documented in the stack README, wired into `pre-backup-export.sh`) and set `kirito.backup: "true"` | Raw copies of live DBs are not backups |
| 12 | No `container_name` collisions; no `depends_on` across stacks | Stacks are independent deployable units |
| 13 | Logging: inherit the daemon default (`local` driver, rotated); per-service overrides MUST be justified in the stack README | SSD protection |

## Validation

CI renders every stack with `docker compose config -q` against
`.env.example`; `deploy.sh` re-renders against real secrets before
`up -d`. A stack that renders only with real values is a CI failure by
design — fix the `.env.example`.
