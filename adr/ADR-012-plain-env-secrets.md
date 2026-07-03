# ADR-012 — Plain .env Secrets (On-Pi Only)

- Status: Accepted
- Date: 2026-07-03

## Context

[ADR-008](ADR-008-secrets-management.md) defines *where* secrets live and
how leakage is prevented. This ADR records the **storage-format decision**:
plain files vs. encrypted-in-Git — decided explicitly by the user in favor
of plain files.

## Decision

Secrets are stored as **plain (unencrypted) files** in
`/srv/homelab/secrets/` on the Pi, protected by filesystem permissions
(`0700`/`0600`), and exist elsewhere **only** inside encrypted Duplicati
backups.

## Consequences (accepted trade-offs)

- **Zero tooling** — no encryption keys to manage in daily operation, no
  decrypt step in `deploy.sh`, nothing to get wrong at 2 a.m.
- **DR depends entirely on the backup chain**: Git alone cannot restore
  secrets. Losing *both* the Pi and access to a backup (or its
  passphrase) means rotating every credential from scratch
  ([RB-005](../runbooks/RB-005-rotate-secrets.md) doubles as the
  from-scratch inventory for that scenario).
- The Duplicati passphrase escrow (password manager + sealed paper,
  checked at quarterly DR drills) is therefore **the** critical
  operational discipline of the platform.
- No secret history/versioning: an accidental overwrite of an `.env` is
  recoverable only from the previous night's backup.

## Upgrade path (improvement #3)

SOPS + age: encrypt the existing `.env` files, commit them to Git, store
the age key in the DR kit, add a decrypt step to `deploy.sh`. Restores
Git-based secret recovery and history. This ADR gets superseded when
adopted.

## Alternatives considered

- **SOPS + age** — recommended by the architecture, declined by user
  decision for operational simplicity; see upgrade path.
- **Encrypted archive of secrets in Git (manual)** — worst of both:
  manual crypto handling with none of SOPS's diff/partial-encryption
  ergonomics.
