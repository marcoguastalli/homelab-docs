# ADR-006 — GitHub Actions with a Self-Hosted Deploy Runner

- Status: Accepted
- Date: 2026-07-03

## Context

The Pi is not reachable from the internet (VPN-only), so a hosted CI
system cannot push deployments to it directly. Code is on GitHub; repos
are public (FOSS). CI must validate every PR; CD must apply merged
changes to the Pi automatically.

## Decision

**GitHub Actions** for all CI/CD, with a split runner model:

- **GitHub-hosted runners**: all validation (PRs process untrusted input),
  releases, scheduled scans and reports.
- **Self-hosted runner on the Pi** (installed by bootstrap as a systemd
  service, running as the unprivileged `homelab` user): deploy workflows
  and backup verification only.

Isolation rules for the self-hosted runner (all mandatory — public repos):

1. Used only by workflows triggering on `push` to `main` — never
   `pull_request`.
2. Actions setting: *Require approval for all outside collaborators*.
3. Deploy jobs bound to the `production` environment.
4. Runner user: `docker` group only, no sudo.

Reusable workflows (`workflow_call`) live in `homelab-ops`; the other
repos are thin callers — one implementation of validate/deploy logic.

## Consequences

- Push-based CD works without exposing the Pi: the runner makes an
  *outbound* connection to GitHub.
- The runner is an attack-surface addition, contained by the rules above.
- Deploy scripts are deliberately runnable without GitHub
  (`deploy.sh` locally) — GitHub down does not mean the platform is
  unmanageable.

## Alternatives considered

- **Pull-based GitOps agent** (systemd timer: `git pull` + `compose up`) —
  purer GitOps, rejected: no deploy logs in the PR context, no health-gate
  → rollback integration with CI status, and a homegrown agent to maintain.
- **Manual SSH deploys** — rejected: CI validates but a human applies;
  drift between `main` and reality is guaranteed eventually.
- **Drone/Woodpecker/Gitea Actions self-hosted** — rejected: another
  service to run *on* the platform it deploys (chicken-and-egg during DR).
