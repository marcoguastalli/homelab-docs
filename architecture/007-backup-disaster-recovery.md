# 007 — Backup & Disaster Recovery

Backup design rationale: [ADR-007](../adr/ADR-007-backup-strategy.md).
Executable procedures: [RB-003](../runbooks/RB-003-restore-single-service.md)
(single service) and [RB-004](../runbooks/RB-004-full-disaster-recovery.md)
(bare metal).

## What is backed up

| Data | Location | In backup? | Why |
|---|---|---|---|
| Compose files, config, scripts, docs | GitHub (5 repos) | Git **is** the backup | — |
| Service state | `/srv/homelab/data/` | ✅ Duplicati | Irreplaceable |
| Secrets (`.env`, `users.yml`, TLS key) | `/srv/homelab/secrets/` | ✅ Duplicati | Exists **nowhere else** ([ADR-012](../adr/ADR-012-plain-env-secrets.md)) |
| Deploy state manifests | `/srv/homelab/state/` | ✅ Duplicati | Speeds recovery |
| Images, containers, OS | — | ❌ | Reproducible from Git + registries |

## Pipeline (nightly, 03:00)

1. **Consistent dumps** — `homelab-ops/scripts/backup/pre-backup-export.sh`
   (cron) writes state manifests; every stateful service defines a dump
   command (`pg_dump`, `sqlite3 .backup`, …) writing into
   `${DATA_ROOT}/<stack>/dumps/`. Raw file copies of live databases are
   not valid backups ([STD-002](../standards/STD-002-compose-conventions.md)).
2. **Duplicati** — AES-256 encrypted, deduplicated backup of
   `data/`, `secrets/`, `state/` → `/mnt/backup` (USB pendrive).
3. **Retention** — 7 daily, 4 weekly, 6 monthly.
4. **Verification (weekly)** — `backup-verification.yml` restores a canary
   file plus one randomly chosen service directory to a temp dir and
   compares checksums. Failure alerts at `critical`.
   *An unverified backup is a hope, not a backup.*

## Objectives

| Objective | Value | Determined by |
|---|---|---|
| RPO | ≤ 24 h | Nightly backup schedule |
| RTO (full rebuild) | ~2 h | Drilled procedure [RB-004](../runbooks/RB-004-full-disaster-recovery.md) |
| RTO (single service) | ~15 min | [RB-003](../runbooks/RB-003-restore-single-service.md) |
| RTO (bad deploy) | ~2 min | Automatic rollback ([009](009-cicd-architecture.md)) |
| RTO (config mistake) | minutes | `git revert` — which *is* a deploy |

## The DR kit (must exist OFF the Pi)

1. GitHub account access (with 2FA recovery codes)
2. **Duplicati passphrase** — password manager **and** sealed paper copy.
   This is the single point of failure of the whole DR story
   ([RB-005](../runbooks/RB-005-rotate-secrets.md) keeps it current).
3. The USB backup pendrive
4. A spare SD card or SSD with Raspberry Pi OS imager access
5. Router admin access (port forward + DHCP DNS settings)

## Documented weaknesses (accepted)

- **No off-site copy** — the pendrive dies with the Pi in fire/theft/surge.
  Fails 3-2-1. Listed as improvement #1: second Duplicati job to Backblaze
  B2 (~$1/month) or a rotated second pendrive stored elsewhere.
- **Passphrase escrow is manual** — no technical control can verify the
  paper copy exists. The quarterly DR drill includes checking it.

## DR drills

- **Quarterly:** tabletop walk-through of RB-004 (are the steps still
  true? does the DR kit exist?). Scheduled as a recurring GitHub issue.
- **Annually:** real restore onto a spare SD card.
  A recovery procedure that has never been executed is fiction.

## Storage migration path (future external USB SSD)

`DATA_ROOT=/srv/homelab/data` is the single variable behind every bind
mount. Migration: mount the new SSD at `/mnt/data` → stop stacks → rsync →
bind-mount or remount at the same path (or flip `DATA_ROOT`) → start
stacks. Near-zero config churn — this is why named volumes were rejected
([STD-002](../standards/STD-002-compose-conventions.md)).
