# RB-004 — Full Disaster Recovery (bare metal → running platform)

- Status: **Draft** — must be validated by the first real drill
- Use when: Pi dead/lost/corrupted. Target RTO ~2 h, RPO ≤ 24 h.
- **Preconditions — the DR kit** ([architecture/007](../architecture/007-backup-disaster-recovery.md)):
  GitHub access (+2FA recovery codes), **Duplicati passphrase**, the USB
  backup pendrive, spare SD/SSD + imager, router admin access.

## Phase 1 — Host (~40 min)

1. Flash Raspberry Pi OS (64-bit Lite) with SSH enabled; boot.
2. LAN sanity: router may briefly need a public DNS for clients while
   Pi-hole is down ([architecture/003](../architecture/003-network-architecture.md)).
3. Provision:

   ```bash
   git clone https://github.com/marcoguastalli/homelab-bootstrap
   cd homelab-bootstrap
   sudo ./bootstrap.sh          # sets static IP, Docker, UFW, mounts, runner
   sudo ./stages/70-verify.sh   # must pass before continuing
   ```

## Phase 2 — Data & secrets (~30–60 min)

1. Plug the backup pendrive (auto-mounts at `/mnt/backup` per fstab).
2. Run the documented one-shot Duplicati restore container (exact command
   in `homelab-bootstrap/docs/RECOVERY.md`) and restore
   `/srv/homelab/secrets` and `/srv/homelab/data` with the escrowed
   passphrase.
3. `chown -R homelab:homelab /srv/homelab && chmod 700 /srv/homelab/secrets`.

## Phase 3 — Platform (~20 min)

Runner was re-registered by bootstrap; either re-run the `deploy.yml`
workflows from GitHub (infrastructure first, then services) **or**
locally, in order:

```bash
sudo -iu homelab
cd /srv/homelab/repos/homelab-ops
./scripts/deploy/deploy.sh homelab-infrastructure networks
./scripts/deploy/deploy.sh homelab-infrastructure traefik    # then authelia, pihole, wireguard
./scripts/deploy/deploy.sh homelab-services monitoring       # then remaining stacks
```

## Phase 4 — Verify (~10 min)

1. `./scripts/checks/smoke-test.sh` — DNS answers, auth redirect works,
   all health checks green.
2. Router: point DHCP DNS back at the Pi; confirm a LAN client resolves
   `home.kirito.com`.
3. Uptime Kuma all green; Grafana shows data; **trigger a manual backup**
   so the cycle restarts tonight.

## Drills

Quarterly tabletop (walk the steps, verify the DR kit exists — including
the paper passphrase); annually for real on spare media. Update this
runbook with every drill finding.
