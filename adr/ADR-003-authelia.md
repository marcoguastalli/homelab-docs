# ADR-003 — Authelia for SSO + 2FA

- Status: Accepted
- Date: 2026-07-03

## Context

Every admin surface must be protected by strong authentication, and many
self-hosted apps ship weak or no auth. One login for the whole platform
(SSO) with 2FA is a hard requirement; the user base is household-scale
(<5 users).

## Decision

**Authelia** as the central authentication portal, integrated via
Traefik's forwardAuth middleware (`authelia@file`).

- Session cookie scoped to `.kirito.com` → SSO across all subdomains.
- Access-control policy is version-controlled in
  `homelab-infrastructure/authelia/config/configuration.yaml`:
  `two_factor` (TOTP) for all admin surfaces, `one_factor` for user apps,
  `bypass` only for the login portal itself
  ([architecture/005](../architecture/005-platform-services.md) has the
  authoritative per-service table).
- **File user backend** (`users.yml`, argon2id hashes, lives in
  `/srv/homelab/secrets/`) — no LDAP.
- SQLite storage; filesystem notifier initially (VPN-only platform, no
  SMTP dependency); upgradeable to SMTP later.
- Services with native auth (e.g. Grafana) keep it **in addition** —
  defense in depth.

## Consequences

- Services with zero built-in auth get full SSO+2FA protection for free.
- File backend = no self-service password reset; acceptable at household
  scale, revisit if users >5.
- Authelia down = nothing behind it reachable. Accepted; it is stateless
  enough to restart fast, monitored, and auto-rolled-back on bad deploys.

## Alternatives considered

- **Authentik** — more features (full OIDC/SAML IdP), rejected: heavier
  (~500 MB+ vs ~50 MB) on an 8 GB budget, larger attack surface, features
  unneeded at this scale.
- **Basic auth per service** — rejected: no SSO, no 2FA, credential
  sprawl, secrets in Traefik labels.
- **Nothing (VPN is the auth)** — rejected: fails the compromised-LAN-
  device threat ([architecture/004](../architecture/004-security-architecture.md)).
