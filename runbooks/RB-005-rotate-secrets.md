# RB-005 — Rotate Secrets

- Status: **Draft** — inventory grows with each new service
- Use when: scheduled rotation, suspected leak, or **any secret found in
  Git** (treat as compromised — deleting the commit is NOT sufficient on
  public repos).

## Secret inventory

| Secret | Location | Rotation procedure |
|---|---|---|
| Per-stack `.env` values | `/srv/homelab/secrets/<stack>.env` | Edit file → redeploy stack (below) |
| Authelia users (`users.yml`) | `/srv/homelab/secrets/authelia/` | `docker run --rm authelia/authelia:<pin> authelia crypto hash generate argon2` → replace hash → restart authelia |
| TLS wildcard key | `/srv/homelab/secrets/tls/` | Re-run `generate-selfsigned.sh` → restart traefik → **re-trust on every device** ([RB-008](RB-008-trust-selfsigned-cert.md)) |
| WireGuard peer keys | wg-easy data | Revoke peer in `vpn.kirito.com` UI → recreate ([RB-007](RB-007-add-vpn-peer.md)) |
| Duplicati passphrase | Duplicati config + **DR kit escrow** | See below — special case |
| GitHub Actions secrets (ntfy token…) | repo Settings → Secrets | Rotate at source → update in GitHub |
| GitHub runner token | auto-managed by runner | Re-register via bootstrap `stages/60-runner.sh` if compromised |

## Standard rotation (per-stack value)

```bash
sudo -iu homelab
vi /srv/homelab/secrets/<stack>.env       # change the value (also at the service side if external)
cd /srv/homelab/repos/homelab-ops
./scripts/deploy/deploy.sh homelab-services <stack>   # recreates containers with new env
```

Verify the service works, then confirm tonight's backup captured the new
file.

## Duplicati passphrase (special case — the DR-kit secret)

Changing it means **old backup versions keep the old passphrase**:

1. Set the new passphrase on the Duplicati job; run a full backup.
2. Update **both** escrow copies (password manager + sealed paper) —
   this step is the entire point.
3. Keep the old passphrase in the password manager, labeled with its
   validity end date, until retention has cycled out all old versions
   (~6 months).
4. Run the weekly verification workflow manually once to confirm.

## After a leak specifically

1. Rotate the leaked secret first, then investigate exposure window.
2. Check access logs where available (GitHub audit log, service logs).
3. Add a gitleaks rule if the pattern wasn't caught; note the incident in
   the repo's CHANGELOG.
