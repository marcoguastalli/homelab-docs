# RB-006 — Update Services (monthly maintenance window)

- Status: **Draft** — finalize when update-report.yml exists
- Cadence: monthly, plus immediate for HIGH/CRITICAL Trivy findings on
  ingress/auth/VPN components ([ADR-014](../adr/ADR-014-manual-update-strategy.md))
- Time: 1–2 h for a typical window

## Steps

1. Open the **available updates** issue (maintained weekly by
   `update-report.yml` in homelab-ops).
2. Snapshot state before touching anything:

   ```bash
   # on the Pi — confirm last night's backup succeeded first
   sudo -iu homelab docker ps --format '{{.Names}}: {{.Image}}' | sort
   ```

3. Order: **services first, infrastructure last** (a broken Traefik
   update at the start ruins the window). Within infrastructure:
   traefik → authelia → pihole → wireguard, one at a time.
4. Per stack worth updating:
   - Read the release notes. Breaking changes / migrations? If a data
     migration is involved, note that **revert is not a rollback plan**
     in the PR and take a manual pre-update backup.
   - PR: `chore(<stack>): bump <image> X → Y` (one stack per PR).
   - CI green → merge → auto-deploy → health check.
   - Verify in the service UI + Grafana/Uptime Kuma before the next one.
5. Skipped updates: comment why on the issue (stays visible next month).
6. Anything broken: `git revert` the merge (redeploys the old pin) —
   unless a data migration ran; then follow the stack README's downgrade
   notes or [RB-003](RB-003-restore-single-service.md).

## Close the window

1. `smoke-test.sh` — full pass.
2. Tag releases in touched repos
   ([STD-006](../standards/STD-006-commit-and-branch-conventions.md)):
   the known-good state marker.
3. Close the updates issue. Check the RAM budget table
   ([architecture/005](../architecture/005-platform-services.md)) if any
   footprint changed materially.
