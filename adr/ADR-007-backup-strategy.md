# ADR-007 — Backup Strategy

- Status: Accepted
- Date: 2026-07-03

## Context

Git holds all configuration, so backup scope is exactly: service state
(`data/`), secrets (`secrets/` — which exist nowhere else per
[ADR-012](ADR-012-plain-env-secrets.md)) and deploy state. The chosen
target hardware is a USB pendrive attached to the Pi (user decision).

## Decision

- **Duplicati**, running as a service stack, performs nightly (03:00)
  AES-256-encrypted, deduplicated backups of `/srv/homelab/{data,secrets,state}`
  to `/mnt/backup` (USB pendrive).
- Retention: 7 daily, 4 weekly, 6 monthly.
- **Consistent dumps before backup**: every stateful service defines a
  dump command (`pg_dump`, `sqlite3 .backup`, …) run by
  `pre-backup-export.sh`; raw copies of live DB files are not valid
  backups ([STD-002](../standards/STD-002-compose-conventions.md)).
- **Weekly automated verification**: canary restore + checksum comparison
  (`backup-verification.yml`); failure alerts at `critical`.
- The Duplicati **passphrase is escrowed off-Pi** in two forms (password
  manager + sealed paper). It is the single point of failure of disaster
  recovery.

## Consequences

- RPO ≤ 24 h; full-rebuild RTO ~2 h
  ([architecture/007](../architecture/007-backup-disaster-recovery.md)).
- **Known accepted weakness: no off-site copy.** Fire/theft/surge that
  takes the Pi and the pendrive together loses all state. This fails
  3-2-1 and is the platform's #1 listed improvement (Backblaze B2 second
  job, ~$1/month, or a rotated second pendrive).
- Pendrive wear/failure risk is mitigated by weekly verification alerts.

## Alternatives considered

- **restic/borg + cron** — excellent tools, rejected (narrowly): Duplicati
  was specified, has a UI for ad-hoc restores, and scheduling/retention
  built in. Swappable later without architectural change — the backup
  *scope and verification* are the architecture; the tool is a detail.
- **Cloud-first backup** — deferred, not rejected: it is improvement #1.
- **RAID / second local disk** — rejected as a *backup* (it isn't one;
  it's availability, a non-goal).
