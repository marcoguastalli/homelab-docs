# RB-001 — Deploy a Service (change an existing one)

- Status: **Draft** — finalize when homelab-ops deploy pipeline exists
- Preconditions: platform running; change already known (config edit, pin bump…)

## Normal path (everything through Git)

1. Branch in the owning repo:

   ```bash
   git switch -c chore/grafana-bump
   ```

2. Edit the stack (compose file, config, pin). Respect
   [STD-002](../standards/STD-002-compose-conventions.md).
3. Commit (conventional), push, open PR. Fill the three PR questions
   (what / which stacks redeploy / rollback plan).
4. Wait for CI green → squash-merge.
5. `deploy.yml` runs on the self-hosted runner: watch the workflow run.
   It deploys only the changed stack, then health-checks it.
6. Verify: service answers on its subdomain; no new alerts; Uptime Kuma
   still green.

## If the deploy fails

- Health-check failure triggers **automatic rollback** to the last good
  SHA — the workflow run shows both the failure and the rollback.
- Fix forward with a new PR, or `git revert` the merge (which is itself
  a deploy).

## Emergency path (GitHub unreachable)

On the Pi:

```bash
sudo -iu homelab
cd /srv/homelab/repos/homelab-ops
./scripts/deploy/deploy.sh homelab-services <stack>
```

`deploy.sh` works without GitHub by design. Reconcile afterwards: make
sure whatever you deployed matches `main` (the nightly validation will
complain if not).
