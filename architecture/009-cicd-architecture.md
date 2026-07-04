# 009 — CI/CD Architecture

Platform choice rationale: [ADR-006](../adr/ADR-006-github-actions.md).

## Pipeline overview

```text
        GITHUB-HOSTED (untrusted-input safe)      SELF-HOSTED on Pi (trusted, main only)
PR ──▶ [yamllint · compose config · gitleaks ──merge──▶ [paths-filter]
        · markdownlint · shellcheck]                        │
                                                            ▼
                                                 [deploy.sh stack A] ─▶ [health ✅]
                                                 [deploy.sh stack B] ─▶ [health ❌]
                                                                           │
                                                                     [rollback.sh]
                                                                           │
                                                                  [alert + red job]
```

Properties: **reproducible** (deploys pin the exact merge SHA),
**idempotent** (unchanged stacks are no-ops), **serialized** (flock on the
Pi), **self-healing on failure** (auto-rollback to last recorded good SHA),
**observable** (every deploy is a workflow run with permanent logs).

## Runner placement — the security-relevant split

| Runner | Runs | Why |
|---|---|---|
| GitHub-hosted | All PR validation, releases, scheduled scans/reports | May process untrusted input (public repos, fork PRs) |
| Self-hosted (on the Pi) | Deploys (`push` to `main` only), backup verification | Needs Docker + filesystem access; isolation rules in [004](004-security-architecture.md) |

## Workflow inventory

Reusable core lives in **homelab-ops**; other repos are thin callers
(one implementation, four consumers — no duplicated CI logic).

| Workflow | Repo | Trigger | Runner | Does |
|---|---|---|---|---|
| `reusable-validate.yml` | ops | `workflow_call` | hosted | yamllint · `docker compose config` per stack (dummy env from `.env.example`) · markdownlint · gitleaks |
| `reusable-deploy.yml` | ops | `workflow_call` | self-hosted | paths-filter → `deploy.sh` per changed stack → `health-check.sh` → rollback on failure |
| `ci.yml` | all 5 | PR + push main | hosted | calls reusable-validate (+ shellcheck/shfmt where bash exists) |
| `deploy.yml` | infrastructure, services | push main | self-hosted | calls reusable-deploy (environment: `production`) |
| `release.yml` | all 5 | tag `v*` | hosted | git-cliff changelog → GitHub Release |
| `nightly-validation.yml` | ops | cron 02:00 | hosted | re-validate `main` of all repos; Trivy scan all pinned images → issue on HIGH/CRITICAL |
| `backup-verification.yml` | ops | cron Sun 04:00 | self-hosted | canary restore + checksum compare; alert on failure |
| `update-report.yml` | ops | cron Mon 06:00 | hosted | outdated pinned images → single GitHub issue, edited in place |
| docs `ci.yml` | docs | PR + push | hosted | markdownlint · link check |

## deploy.sh contract (implemented in homelab-ops)

`deploy.sh <repo> <stack>` — idempotent, locked:

1. Acquire `/srv/homelab/state/deploy.lock` (flock) — serializes deploys.
2. `git fetch && checkout <exact SHA>` in `/srv/homelab/repos/<repo>`.
3. Assert `/srv/homelab/secrets/<stack>.env` exists (actionable error if not).
4. `docker compose --env-file … config -q` — final render with real values
   (CI validated with dummies).
5. `docker compose pull` (pinned tags → pulls only when the pin changed).
6. `docker compose up -d --remove-orphans`.
7. `health-check.sh <stack>`: wait for `healthy`, then HTTP check through
   Traefik. Timeout → non-zero → workflow runs `rollback.sh` to the last
   good SHA.
8. Record deployed SHA in `/srv/homelab/state/<stack>.deployed`.

Ordering guard: refuses to deploy a service when `net_proxy` does not
exist (infrastructure must deploy first).

## Testing strategy (what "tested" means here)

| Stage | Test | Where |
|---|---|---|
| Static | yamllint, markdownlint, shellcheck, shfmt | PR CI |
| Structural | `docker compose config` per stack | PR CI |
| Security | gitleaks (PR) · Trivy (nightly) | CI |
| Deploy-time | container healthcheck + HTTP via Traefik | deploy gate |
| Runtime | Prometheus alerts + Uptime Kuma | continuous |
| Periodic | nightly re-validation, weekly backup verify | scheduled |
| Host contract | bootstrap `stages/70-verify.sh` | after provisioning |
| DR | quarterly tabletop, annual real restore | calendar ([007](007-backup-disaster-recovery.md)) |
