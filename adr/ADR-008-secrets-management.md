# ADR-008 — Secrets Management

- Status: Accepted
- Date: 2026-07-03

## Context

Secrets (service credentials, API tokens, Authelia `users.yml`, TLS
private key, Duplicati passphrase) must never appear in public Git repos,
must survive disaster recovery, and must be simple enough for a solo
maintainer to handle correctly every time.

## Decision

- Secrets live **only** on the Pi in `/srv/homelab/secrets/`
  (dir `0700`, files `0600`, owner `homelab`) and in encrypted backups.
  Storage-format decision: [ADR-012](ADR-012-plain-env-secrets.md).
- One `.env` file per stack: `/srv/homelab/secrets/<stack>.env`,
  referenced via `env_file:` in compose. Non-env secrets (TLS key,
  `users.yml`) live in subdirectories, bind-mounted read-only.
- Every stack commits a complete **`.env.example`** with dummy values —
  the documented contract of what the stack needs; CI validates compose
  rendering against it.
- **Enforcement**: gitleaks runs in every repo's CI on every PR *and* as a
  pre-commit hook. A leaked secret is treated as compromised and rotated
  ([RB-005](../runbooks/RB-005-rotate-secrets.md)) — deleting the commit
  is not sufficient on public repos.
- GitHub Actions secrets are limited to what CI itself needs (e.g. ntfy
  token for alerts); deployment secrets never transit GitHub — the runner
  reads them locally from `/srv/homelab/secrets/`.

## Consequences

- Simple, auditable, zero extra tooling; `deploy.sh` fails fast with an
  actionable message when a stack's `.env` is missing.
- Disaster recovery of secrets depends entirely on the Duplicati backup +
  escrowed passphrase — the accepted trade-off detailed in
  [ADR-012](ADR-012-plain-env-secrets.md).

## Alternatives considered

- **SOPS + age (encrypted secrets in Git)** — the strongest alternative:
  Git-based secret recovery and history. Declined by explicit user
  decision in favor of operational simplicity; listed as improvement #3.
  Upgrade path: encrypt the existing `.env` files, commit, teach
  `deploy.sh` to decrypt.
- **Docker secrets** — designed for Swarm; with plain compose they are
  file mounts with extra steps.
- **Vault/OpenBao** — a stateful, unsealed-key-managing service to keep a
  household platform's secrets: operational overkill and a DR
  chicken-and-egg.
