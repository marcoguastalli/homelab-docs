# ADR-009 — Versioning & Branching

- Status: Accepted
- Date: 2026-07-03

## Context

One maintainer, five repos, continuous deployment from `main`. The
process must add safety without adding ceremony that a solo maintainer
will inevitably start skipping.

## Decision

- **Trunk-based development**: short-lived `feat/`, `fix/`, `docs/`,
  `chore/` branches; squash-merge to protected `main`; no develop or
  release branches.
- **PRs required with green CI, self-merge allowed.** The PR template
  (what changed / which stacks redeploy / rollback plan) is the review.
- **Conventional Commits** feed automated changelogs.
- **SemVer per repository, independent.** Major = breaking cross-repo
  contract; minor = new capability; patch = fixes and pin bumps.
- Full normative rules: [STD-006](../standards/STD-006-commit-and-branch-conventions.md);
  strategy details: [architecture/008](../architecture/008-git-strategy.md).

## Consequences

- `main` = deployed reality; drift is structurally impossible while CI/CD
  works, and detectable (nightly validation) when it doesn't.
- Honest process: no pretend reviews from a nonexistent team, but the
  merge gate (CI + written rollback plan) is real.
- Independent SemVer means "platform version" is a *set* of tags — that
  is exactly what release tagging after maintenance windows captures
  ([ADR-010](ADR-010-release-strategy.md)).

## Alternatives considered

- **GitFlow** — rejected: designed for scheduled releases and parallel
  version support; pure ceremony for continuous solo deployment.
- **Direct pushes to main (no PRs)** — rejected: loses CI gating,
  the written rollback plan, and the audit trail; the marginal speed gain
  is not worth it even solo.
- **Lockstep versioning across repos** — rejected: forces meaningless
  version bumps in untouched repos.
