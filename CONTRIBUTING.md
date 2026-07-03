# Contributing

This is a personal FOSS homelab project, but it follows an enterprise-style
workflow deliberately — the process is part of the product.

## Workflow

1. Branch from `main`: `docs/<topic>`, `fix/<topic>` or `chore/<topic>`.
2. Commit using [Conventional Commits](https://www.conventionalcommits.org/):
   `docs:`, `fix:`, `chore:`.
3. Open a pull request. CI must pass (markdownlint + link check).
4. Merge (squash) after green CI. `main` is protected: no direct pushes,
   no force pushes.

## What goes where

| You want to… | Do this |
|---|---|
| Propose an architectural change | Draft in `decisions/proposals/`, discuss, then promote to an ADR |
| Record an accepted decision | Add `adr/ADR-NNN-<slug>.md` (next free number, MADR format) |
| Change how something is built | Edit the relevant `architecture/` document + reference the ADR |
| Change a convention | Edit the `standards/` document via PR — standards are normative |
| Fix or add an operational procedure | Edit `runbooks/` — runbook fixes discovered during incidents are part of the incident resolution |

## Rules

- ADRs are **append-only**: never edit an accepted ADR, supersede it with a
  new one and cross-link both.
- Secrets, hostnames of other people, and personal data never appear here.
- Diagrams: commit the editable source in `diagrams/` alongside the exported
  `.svg`.
- Writing style and structure: see
  [STD-004](standards/STD-004-documentation-standards.md).
