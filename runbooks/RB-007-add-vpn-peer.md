# RB-007 — Add a VPN Peer

- Status: **Draft** — finalize against the deployed wg-easy version
- Use when: enrolling a new device (phone, laptop) for remote access

## Steps

1. From a device already on LAN/VPN: `https://vpn.kirito.com`
   (Authelia two_factor).
2. Create the peer, named for the device: `marco-phone`, `marco-laptop` —
   one peer per device, never shared configs.
3. Confirm pushed client DNS is the Pi's LAN IP (Pi-hole) — this is what
   makes `*.kirito.com` resolve remotely and keeps ad-blocking on VPN.
4. Install: WireGuard app → scan the QR (mobile) or download the
   `.conf` (laptop). Handle the config as a secret — it contains the
   private key; delete downloaded copies after import.
5. Trust the TLS cert on the new device: [RB-008](RB-008-trust-selfsigned-cert.md).
6. Test **from off-LAN** (disable Wi-Fi on mobile): connect VPN → open
   `home.kirito.com` → confirm login via Authelia.

## Housekeeping

- Review peers during the monthly window; delete peers for retired
  devices immediately (a lost device's peer = [RB-005](RB-005-rotate-secrets.md)
  WireGuard row).
- Peer data is part of the wireguard stack's state under
  `${DATA_ROOT}` — covered by the nightly backup.
