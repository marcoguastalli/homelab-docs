# STD-005 — Shell Coding Standards

Normative for all bash in homelab-bootstrap and homelab-ops. Context:
[ADR-013](../adr/ADR-013-bash-bootstrap.md) — bash is acceptable *because*
these rules are enforced.

## Non-negotiables

1. Shebang `#!/usr/bin/env bash`; `set -Eeuo pipefail` immediately after.
2. **shellcheck-clean** (no suppressions without an inline justification
   comment) and **shfmt** formatted (`-i 2 -bn -ci`) — both enforced in CI.
3. **Idempotent**: every script and every bootstrap stage checks state
   before changing it and is safe to re-run. "Run once only" scripts are
   banned.
4. Error trap from the shared library reporting script name, line and
   failed command:

   ```bash
   source "$(dirname "${BASH_SOURCE[0]}")/../lib/common.sh"  # sets traps, log fns, lock helpers
   ```

5. Mutating operations on shared state MUST take the flock-based lock
   (`common.sh`) — e.g. deploys serialize on `/run/homelab-deploy.lock`.

## Style

| Rule | Example |
|---|---|
| Quote every expansion | `"$stack"`, `"${files[@]}"` |
| `local` for all function variables; UPPER_SNAKE only for exported/env | `local stack="$1"` |
| Functions verb-first; `main "$@"` entry point at the bottom | `deploy_stack() { … }` |
| Prefer `[[ ]]`, `$(…)`, arrays over `[ ]`, backticks, word-splitting | — |
| Fail with actionable messages on stderr, correct exit codes | `die "missing /srv/homelab/secrets/${stack}.env — create it from .env.example"` |
| No `eval`; no parsing `ls`; `mktemp` + trap-cleanup for temp files | — |
| Usage/`--help` output for every user-facing script | — |

## Compatibility

Target: bash 5.x on Raspberry Pi OS (the platform) **and** bash 3.2 on
macOS for anything meant to run on the operator's machine — scripts in
homelab-ops SHOULD state which environment they target in their header
comment.

## Testing

Scripts with non-trivial logic SHOULD have BATS tests; the bootstrap
contract is tested by `stages/70-verify.sh` (assertions, not descriptions).
