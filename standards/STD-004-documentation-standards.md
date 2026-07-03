# STD-004 — Documentation Standards

Normative for all five repositories.

## Where prose lives

| Content | Location |
|---|---|
| Architecture, decisions, runbooks, standards | **homelab-docs only** |
| How to use *this repo/stack/script* | `README.md` next to the code |
| Why a change was made | The PR + conventional commit; decisions worth keeping get promoted to an ADR |

Code repos MUST NOT grow their own architecture documents — they link
here. Duplicated prose diverges; links don't.

## Required files per repository

`README.md`, `LICENSE` (MIT), `CHANGELOG.md` (Keep-a-Changelog,
append-only), `CONTRIBUTING.md`. Stack directories and non-obvious
subdirectories SHOULD carry a focused `README.md`.

## Document types in this repo

| Type | Prefix | Mutability | Voice |
|---|---|---|---|
| Architecture | `NNN-` | Living — updated as the platform evolves | Descriptive: *what and how it is* |
| ADR | `ADR-NNN-` | **Immutable** once accepted; supersede, never edit | Past decision: context → decision → consequences → alternatives |
| Runbook | `RB-NNN-` | Living — fixing them is part of incident resolution | Imperative steps, copy-pasteable commands, explicit preconditions |
| Standard | `STD-NNN-` | Living, normative | RFC-2119: MUST / SHOULD / MAY |
| Proposal | `decisions/proposals/` | Draft until promoted to ADR or rejected (kept) | Open question |

## Writing rules

1. Lead with purpose: first paragraph says what the document is for.
2. Tables for enumerable facts; prose for reasoning; ASCII or committed
   SVG for diagrams (source file committed alongside in `diagrams/`).
3. Every claim about behavior points at code (repo-relative link) or at
   an ADR — undocumented rationale rots.
4. Commands in runbooks MUST be copy-pasteable (no `<placeholders>`
   inside command blocks; define shell variables first).
5. Relative links within the repo; full URLs across repos.
6. Linted by markdownlint (config at repo root) + lychee link check in CI.
7. Dates in ISO 8601 (`2026-07-03`); absolute dates, never "last month".
