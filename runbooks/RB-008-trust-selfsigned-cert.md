# RB-008 — Trust the Self-Signed Certificate (per device)

- Status: **Draft** — verify steps per OS version when the cert exists
- Context: [ADR-011](../adr/ADR-011-self-signed-tls.md). One 10-year
  wildcard cert for `*.kirito.com` — each device does this **once**.

## Get the certificate onto the device

Easiest transport: browse to any `https://*.kirito.com` page → export the
certificate from the browser warning screen, or copy
`/srv/homelab/secrets/tls/kirito.com.crt` (the public cert — **never the
`.key`**) via AirDrop/USB/email-to-self.

## Per OS

### macOS

1. Double-click the `.crt` → Keychain Access adds it (System keychain).
2. Keychain Access → find `*.kirito.com` → Get Info → Trust →
   **Always Trust** (When using this certificate).
3. Restart the browser. Firefox: also Settings → Privacy → Certificates →
   Import (Firefox uses its own store), or set
   `security.enterprise_roots.enabled` to `true`.

### iOS

1. Open the `.crt` (AirDrop/Files) → profile downloaded.
2. Settings → General → VPN & Device Management → install the profile.
3. **Settings → General → About → Certificate Trust Settings → enable
   full trust** for the cert — the step everyone forgets.

### Android

1. Settings → Security → More → Encryption & credentials →
   Install a certificate → **CA certificate** → accept the warning.
2. Note: apps that opt out of user CAs may still refuse — the known
   limitation from [ADR-011](../adr/ADR-011-self-signed-tls.md).

### Linux (Debian-family)

```bash
sudo cp kirito.com.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

Chromium/Firefox keep their own stores — import via browser settings if
warnings persist.

## Verify

`https://home.kirito.com` shows a padlock, no warning. CLI check:

```bash
curl -sSI https://home.kirito.com >/dev/null && echo trusted
```

## When the cert is regenerated

All devices must repeat this. Avoid regenerating casually — that is why
the validity is 10 years ([RB-005](RB-005-rotate-secrets.md), TLS row).
