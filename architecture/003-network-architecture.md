# 003 — Network Architecture

## Address plan

| Network | Range | Notes |
|---|---|---|
| LAN | `192.168.1.0/24` | Placeholder — adjust to the real LAN in bootstrap `config/defaults.env` |
| Pi (static) | `192.168.1.10` | Set by bootstrap; DHCP reservation on the router recommended as belt-and-braces |
| WireGuard | `10.8.0.0/24` | VPN clients; DNS pushed = Pi-hole |
| Docker `net_proxy` | `172.20.0.0/24` | Traefik ↔ HTTP services |
| Docker `net_monitoring` | `172.21.0.0/24` | Prometheus ↔ scrape targets |
| Docker `<stack>_internal` | auto-assigned | Per-stack private networks (app ↔ db) |

## Exposure model

**VPN-only.** The router forwards exactly one port: **UDP 51820** to the Pi.
Nothing else, no exceptions without an ADR. All HTTP(S) access — LAN or
remote — terminates at Traefik on the Pi.

## DNS (split-horizon)

```text
client asks: grafana.kirito.com
    │
    ▼
Pi-hole  (LAN DNS via router DHCP; VPN DNS via WireGuard client config)
    ├─ *.kirito.com   → 192.168.1.10   (dnsmasq address rule, version-controlled)
    └─ everything else → upstream resolver (ad-block filtered)
```

- The wildcard rule lives in
  `homelab-infrastructure/pihole/config/99-kirito.conf`:
  `address=/kirito.com/192.168.1.10`. It is Git-managed, **not** clicked
  into the Pi-hole UI.
- `kirito.com` is an unregistered placeholder domain; because DNS is
  split-horizon and the platform is VPN-only, it never needs to resolve
  publicly. See [ADR-011](../adr/ADR-011-self-signed-tls.md) for the TLS
  consequence.

### The bootstrap paradox, avoided

The Pi itself does **not** use Pi-hole for its own resolution. If Docker is
down, Pi-hole is down — and the Pi could not resolve `github.com` to fix
it. Bootstrap points the host's own resolver at the router/upstream
directly.

### Household risk: Pi down = LAN DNS down

Default stance: Pi-hole is the only DHCP-advertised DNS server (full
ad-blocking, no leaks). Documented alternative: hand out a public resolver
as secondary DNS, trading blocking completeness for resilience. Choose per
household; the default is Pi-hole-only. Recovery implications in
[RB-004](../runbooks/RB-004-full-disaster-recovery.md).

## Docker networks

Created once by `homelab-infrastructure/networks/apply.sh` (the script
stack convention deploy.sh expects) and declared `external: true` in
every compose file.

| Network | Members | Purpose |
|---|---|---|
| `net_proxy` | Traefik + every HTTP-exposed service | The only ingress path |
| `net_monitoring` | Prometheus + scrape targets | Metrics without joining `net_proxy` |
| `<stack>_internal` | One stack's containers only | DB/app isolation; never `external` |

Rationale for a shared proxy network over per-service networks:
[ADR-004](../adr/ADR-004-docker-networking.md). Key point: nothing
sensitive listens on `net_proxy` — databases and internal components live
on `_internal` networks.

## Host firewall

UFW default-deny inbound, plus matching rules in the `DOCKER-USER` iptables
chain because **Docker's published ports bypass UFW**
([ADR-004](../adr/ADR-004-docker-networking.md), the classic homelab hole).

| Port | Proto | Allowed from | Service |
|---|---|---|---|
| 22 | tcp | LAN only | SSH (key-only) |
| 53 | tcp+udp | LAN + VPN | Pi-hole |
| 80, 443 | tcp | LAN + VPN | Traefik (80 only redirects to 443) |
| 51820 | udp | any | WireGuard |

Everything else: deny. Rules are installed by bootstrap stage
`30-firewall.sh` and asserted by stage `70-verify.sh`.
