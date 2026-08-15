# OPNsense Firewall Rules — NETS1037 Project 2

> **Note:** The OPNsense lab VM used to build and test this ruleset is no longer
> available, so no live rule export (XML/CSV) is included. The table below is a
> reconstructed summary of every rule actually implemented and tested during the
> project, in the order they were evaluated.

## Rule ordering note (critical)

Block rules **must** be placed above the default allow rule. Rules positioned below
a default allow are silently ignored by pf/OPNsense's first-match rule evaluation —
confirmed as a bug during testing where blocks appeared configured but had no effect.

## Rules (in evaluation order)

| # | Action | Interface | Protocol | Source | Destination | Port | Purpose |
|---|--------|-----------|----------|--------|-------------|------|---------|
| 1 | Redirect (NAT) | LAN | TCP | any | any | 80, 443 | Transparent redirect to Squid |
| 2 | Redirect (NAT) | LAN | UDP | any | any | 53 | Force DNS through Unbound |
| 3 | Block | LAN | UDP | any | any | 443 | Block QUIC/HTTP-3 |
| 4 | Block | LAN | UDP | any | any | 853 | Block DoT |
| 5 | Block | LAN | TCP/UDP | any | DoH_provider_IPs (alias) | 443 | Block known DoH resolvers |
| 6 | Block | LAN | TCP | any | any | 9001 | Block Tor OR port |
| 7 | Block | LAN | TCP/UDP | any | Tor_exit_guard_nodes (alias) | any | Block known Tor nodes |
| 8 | Block | LAN | TCP/UDP | any | VPN_providers (alias) | any | Block known VPN provider infra |
| 9 | Block | LAN | UDP | any | any | 51820 | Block WireGuard |
| 10 | Block | LAN | UDP | any | any | 1194 | Block OpenVPN |
| 11 | Block | LAN | UDP | any | any | 500, 4500 | Block IKEv2/IPsec |
| 12 | Allow | LAN | any | any | any | any | Default allow (must stay LAST) |

## Aliases used

- `DoH_provider_IPs` — Cloudflare (1.1.1.1, 1.0.0.1), Google (8.8.8.8, 8.8.4.4), etc.
- `Tor_exit_guard_nodes` — pulled from public Tor node list, static snapshot
- `VPN_providers` — known infrastructure ranges/domains for tested VPN providers
