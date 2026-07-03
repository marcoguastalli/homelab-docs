# 004 — Security Architecture

## Threat model (honest version)

With VPN-only exposure, the realistic threats are:

| # | Threat | Addressed by |
|---|---|---|
| 1 | Malicious or compromised container image | Layered container hardening, pinned tags, weekly Trivy scans, no published ports |
| 2 | Compromised device on the LAN | Authelia SSO + 2FA on every admin surface, TLS everywhere, SSH key-only |
| 3 | Supply-chain attack via image update | Manual updates ([ADR-014](../adr/ADR-014-manual-update-strategy.md)) with pinned tags + vulnerability reports |
| 4 | Physical loss/theft of the Pi | Encrypted backups; **no full-disk encryption** — accepted trade-off, see below |
| 5 | Compromise via CI (public repos + self-hosted runner) | Runner isolation rules, see below |

Internet-side attacks are out of scope by construction: the only listening
surface is WireGuard, which does not respond to unauthenticated packets.

## Layered controls

| Layer | Control |
|---|---|
| Network edge | Router forwards **only** UDP 51820. No exceptions. |
| Host firewall | UFW default-deny + `DOCKER-USER` chain rules ([003](003-network-architecture.md)) |
| Host access | SSH key-only, root login disabled, fail2ban, OS **security** updates unattended (the one exception to manual updates — CVE latency on the OS is unacceptable) |
| Ingress | Single entry point (Traefik), TLS everywhere, rate limiting via `secured@file` |
| AuthN/AuthZ | Authelia SSO; `two_factor` on all admin surfaces ([ADR-003](../adr/ADR-003-authelia.md)) |
| Containers | `no-new-privileges`, never `privileged`, mem/CPU limits, `exposedByDefault: false`, minimal mounts |
| Docker socket | Mounted read-only and **only** where unavoidable: Traefik, cAdvisor, Dockge (read-only role, [ADR-015](../adr/ADR-015-dockge-read-only.md)) |
| Secrets | Never in Git — gitleaks in every repo's CI; `/srv/homelab/secrets` is `0700`, files `0600` ([ADR-008](../adr/ADR-008-secrets-management.md)) |
| Supply chain | Pinned image tags; weekly Trivy scan of every pinned image → GitHub issue on HIGH/CRITICAL |

## Self-hosted runner isolation (critical: repos are public)

Fork PRs must never execute code on the Pi. All of the following are
mandatory and verified during setup ([ADR-006](../adr/ADR-006-github-actions.md)):

1. The self-hosted runner is used **only** by deploy workflows, which
   trigger **only** on `push` to `main` — never on `pull_request`.
2. All PR validation runs on GitHub-hosted runners.
3. Repo Actions setting: *Require approval for all outside collaborators*.
4. Deploy jobs are bound to the `production` GitHub environment; the runner
   is not reachable from other workflow contexts.
5. The runner runs as the unprivileged `homelab` user — `docker` group
   membership only, no sudo.

## Accepted trade-offs

- **No full-disk encryption.** Headless FDE on a Pi means every reboot
  needs manual key entry (or a network unlock service that becomes its own
  failure mode). Mitigation: nothing on the SSD is secret except
  `/srv/homelab/secrets` (small, mode-restricted) — and physical theft is
  mitigated by encrypted off-Pi awareness of what to rotate
  ([RB-005](../runbooks/RB-005-rotate-secrets.md)).
- **Self-signed TLS** ([ADR-011](../adr/ADR-011-self-signed-tls.md)):
  devices must trust the CA once; non-browser clients need OS-level trust.
- **Secrets exist only on the Pi + backups**
  ([ADR-012](../adr/ADR-012-plain-env-secrets.md)): losing both the Pi and
  the backup passphrase means full manual secret rotation.

## Firewall rule documentation

The authoritative rule set is code:
`homelab-bootstrap/stages/30-firewall.sh`. The table in
[003-network-architecture.md](003-network-architecture.md) is the
human-readable mirror and MUST be updated in the same PR as any rule
change.
