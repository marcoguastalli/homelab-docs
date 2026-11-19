# RB-009 — Re-enable a Disabled Scheduled Workflow

- Status: **Draft** — leaves Draft the first time it is executed
  successfully end-to-end.
- Context: [ADR-006](../adr/ADR-006-github-actions.md). GitHub
  automatically disables any workflow triggered only by `schedule:` after
  **60 days with no push activity to the repo**. This is expected, not a
  bug — while a repo is quiet (nothing to fix, nothing to deploy), its
  cron jobs go stale on their own.

## Which workflows this applies to

Any workflow with a `schedule:` trigger and no other event that would
naturally keep firing. Currently in `homelab-ops`:

- `nightly-validation.yml` — daily CI-status + Trivy scan
- `backup-verification.yml` — weekly canary restore
- `update-report.yml` — weekly outdated-image report

All three also carry `workflow_dispatch:`, so they can be run manually
even while disabled — no need to re-enable just to do a one-off check.

## Re-enable

1. GitHub → the repo → **Actions** tab.
2. Left sidebar → click the disabled workflow's name (shown with a
   "This scheduled workflow is disabled" note).
3. Click **Enable workflow** at the top of the run list.

No code change, no PR — this is a repo setting, not a file edit. Re-enabling
resets nothing structural; it just resumes the cron trigger, and it will
auto-disable again after another 60 days of repo inactivity if nothing is
pushed.

## Verify

Next scheduled run appears in the Actions run history at (or shortly
after) the next cron tick, or trigger it immediately via **Run workflow**
(`workflow_dispatch`) on the same page.

## When this is expected vs. a problem

- **Expected**: the homelab is idle (no commits, nothing broken) — the
  monitoring these jobs exist for has nothing to report anyway. Leaving
  them disabled and re-enabling on demand is an accepted tradeoff, not a
  gap to fix.
- **Investigate further**: a workflow disables itself while the platform
  is under active use (frequent commits elsewhere in the same repo) — a
  push to a *different* homelab-\* repo does not reset another repo's
  60-day clock, since the check is per-repo activity, not per-platform.
