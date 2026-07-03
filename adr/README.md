# Architecture Decision Records

ADRs record **accepted** decisions and are **immutable**: never edit one,
supersede it with a new one and cross-link both. Drafts and rejected
alternatives live in [../decisions/](../decisions/README.md).

## Format ([MADR](https://adr.github.io/madr/)-style)

```markdown
# ADR-NNN — Title

- Status: Accepted | Superseded by ADR-MMM
- Date: YYYY-MM-DD

## Context
## Decision
## Consequences (incl. trade-offs accepted)
## Alternatives considered
```

## Index

| ADR | Title | Status |
|---|---|---|
| [ADR-001](ADR-001-repository-boundaries.md) | Repository boundaries | Accepted |
| [ADR-002](ADR-002-traefik.md) | Traefik as sole ingress | Accepted |
| [ADR-003](ADR-003-authelia.md) | Authelia for SSO + 2FA | Accepted |
| [ADR-004](ADR-004-docker-networking.md) | Docker networking model | Accepted |
| [ADR-005](ADR-005-monitoring-stack.md) | Monitoring stack | Accepted |
| [ADR-006](ADR-006-github-actions.md) | GitHub Actions + self-hosted runner | Accepted |
| [ADR-007](ADR-007-backup-strategy.md) | Backup strategy | Accepted |
| [ADR-008](ADR-008-secrets-management.md) | Secrets management | Accepted |
| [ADR-009](ADR-009-versioning.md) | Versioning & branching | Accepted |
| [ADR-010](ADR-010-release-strategy.md) | Release strategy | Accepted |
| [ADR-011](ADR-011-self-signed-tls.md) | Self-signed TLS | Accepted |
| [ADR-012](ADR-012-plain-env-secrets.md) | Plain .env secrets | Accepted |
| [ADR-013](ADR-013-bash-bootstrap.md) | Bash bootstrap | Accepted |
| [ADR-014](ADR-014-manual-update-strategy.md) | Manual update strategy | Accepted |
| [ADR-015](ADR-015-dockge-read-only.md) | Dockge read-only | Accepted |
