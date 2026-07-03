# Decisions — proposals and their lifecycle

[ADRs](../adr/README.md) record **accepted** decisions and are immutable.
This directory holds everything *before* that point — including rejected
ideas, which are kept because "why not X" is knowledge that ADRs alone
lose.

## Lifecycle

```text
idea ──▶ proposals/PROP-NNN-<slug>.md (status: Draft)
              │
              ├─ accepted ──▶ promoted to adr/ADR-NNN (proposal updated:
              │               status: Promoted → ADR-NNN, kept here)
              └─ rejected ──▶ status: Rejected + one-paragraph reason, kept here
```

## Proposal format

```markdown
# PROP-NNN — Title

- Status: Draft | Promoted → ADR-NNN | Rejected
- Date: YYYY-MM-DD

## Problem
## Options considered (with trade-offs)
## Leaning / open questions
```

## When is a proposal required?

- Anything breaking a cross-repo contract
  ([architecture/002](../architecture/002-repository-model.md))
- Adopting a heavy service
  ([architecture/005](../architecture/005-platform-services.md))
- Changing an accepted-risk posture (TLS, secrets format, backup target,
  update strategy)

Small reversible choices don't need this — a PR description is enough.

## Current proposals

None yet. Candidates already identified as future improvements: off-site
backup, real domain + Let's Encrypt, SOPS+age, Renovate
(see [architecture/000](../architecture/000-architecture-vision.md),
accepted risks).
