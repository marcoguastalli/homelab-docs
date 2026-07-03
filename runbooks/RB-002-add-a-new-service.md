# RB-002 — Add a New Service

- Status: **Draft** — finalize when homelab-services `_template/` exists
- Preconditions: service chosen; image + arm64 support verified; RAM
  footprint estimated against the budget in
  [architecture/005](../architecture/005-platform-services.md)

## Decision gate first

- Heavy service (Immich, Ollama, Jellyfin, Nextcloud)? → ADR required
  before proceeding ([architecture/005](../architecture/005-platform-services.md)).
- New subdomain name: functional, not product-named
  ([STD-001](../standards/STD-001-naming-conventions.md)).

## Steps

1. In `homelab-services`:

   ```bash
   git switch -c feat/add-<stack>
   cp -r _template <stack>
   ```

2. Edit `compose.yaml`: image pin, `mem_limit`, healthcheck, port label,
   subdomain, auth middleware per policy. Checklist =
   [STD-002](../standards/STD-002-compose-conventions.md) rules table.
3. Write `.env.example` (complete, dummy values) and the stack `README.md`
   (incl. consistent-dump command if stateful).
4. **Docs, same PR discipline:** add the service row to
   [architecture/005](../architecture/005-platform-services.md) + RAM
   budget update; add the Authelia access-control rule
   (homelab-infrastructure PR if policy class is new).
5. On the Pi, create the secrets file **before merging**:

   ```bash
   sudo -iu homelab
   install -m 600 /dev/null /srv/homelab/secrets/<stack>.env
   vi /srv/homelab/secrets/<stack>.env    # fill from .env.example
   mkdir -p /srv/homelab/data/<stack>
   ```

6. PR → CI green → merge → auto-deploy → health check.
7. Post-deploy: trust check in browser, add an Uptime Kuma monitor,
   confirm it appears on Homepage, confirm Prometheus targets if it
   exports metrics.
8. If stateful: verify the next nightly backup includes
   `${DATA_ROOT}/<stack>` (Duplicati job log), and that
   `pre-backup-export.sh` runs its dump.
