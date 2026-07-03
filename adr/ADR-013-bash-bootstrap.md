# ADR-013 — Bash Bootstrap

- Status: Accepted
- Date: 2026-07-03

## Context

Host provisioning (OS hardening, Docker, firewall, mounts, users, CI
runner) must be reproducible and idempotent so that disaster recovery is
"flash OS, run one command". Candidates: Ansible or plain bash. The user
chose bash.

## Decision

**Plain bash**, structured to earn back what a config-management tool
would have provided:

- Numbered, independently runnable **stages** (`stages/NN-*.sh`), each
  idempotent — checks state before acting, safe to re-run.
- One entry point (`bootstrap.sh`) executing stages in lexical order;
  gaps in numbering allow insertion.
- `set -Eeuo pipefail` + shared error traps everywhere
  ([STD-005](../standards/STD-005-shell-coding-standards.md)).
- **`stages/70-verify.sh` is the contract test**: asserts every promise
  bootstrap makes to the other repos (Docker running, mounts present,
  firewall enforcing, runner active). Bootstrap "passes" only if verify
  passes — this substitute for Ansible's convergence model is what makes
  bash acceptable.
- CI: shellcheck + shfmt on every PR.

## Consequences

- No Python/Ansible dependency on the Pi or the operator's machine;
  the scripts are readable by anyone who knows shell.
- Idempotency is **discipline, not guarantee** — Ansible modules give it
  for free; here every stage author must implement check-before-act.
  Mitigated by the verify stage, shellcheck, and the standards doc.
- Complex conditional logic (multi-host, OS variants) would outgrow bash —
  accepted: one host, one OS. A second node is the trigger to revisit
  (likely superseding this ADR with Ansible).

## Alternatives considered

- **Ansible** — the architecture's original recommendation: idempotent
  modules, declarative inventory, dry-run. Declined by user decision;
  revisit at >1 host.
- **Cloud-init / NixOS** — powerful but wrong ecosystem fit for Raspberry
  Pi OS and the stated maintainability-by-one-person goal.
