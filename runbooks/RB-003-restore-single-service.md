# RB-003 — Restore a Single Service

- Status: **Draft** — finalize against the real Duplicati job
- Use when: one service's data is corrupted/lost, platform otherwise fine
- Expected time: ~15 minutes

## Steps

1. Stop the stack:

   ```bash
   sudo -iu homelab
   cd /srv/homelab/repos/homelab-services/<stack>
   docker compose down
   ```

2. Preserve the bad state (cheap insurance):

   ```bash
   mv /srv/homelab/data/<stack> /srv/homelab/data/<stack>.broken.$(date +%F)
   ```

3. Restore from Duplicati — `backup.kirito.com` → Restore → pick the
   version (usually last night) → target the original path
   `/srv/homelab/data/<stack>`. Passphrase: from the DR kit if not cached.
4. For DB-backed services: prefer restoring the **dump**
   (`${DATA_ROOT}/<stack>/dumps/`) into a fresh data dir over raw files —
   the stack README documents its restore command.
5. Start and verify:

   ```bash
   docker compose up -d
   ../homelab-ops/scripts/checks/health-check.sh <stack>
   ```

6. Check the service's own UI/data sanity, then delete
   `<stack>.broken.*` after a few days.

## Afterwards

If the cause was a bad deploy or config: fix via Git. If this runbook had
a wrong/missing step: fixing it is part of closing the incident
([architecture/011](../architecture/011-operations-model.md)).
