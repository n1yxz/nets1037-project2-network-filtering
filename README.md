# NETS1037 — Project 2: Transparent Network Filtering Gateway

> **Note on configs:** The lab VMs (OPNsense gateway + client) used to build and test
> this project are no longer available, so the files under `configs/` are reconstructed
> from the actual build process rather than live exports. They accurately represent the
> rules, ACLs, and settings implemented and tested during the project, but are written
> up manually rather than pulled directly from the running system.

A transparent filtering gateway built on OPNsense, demonstrating traffic interception,
domain/DNS filtering, HTTPS inspection, intrusion detection, and bypass-resistance
testing against common circumvention techniques (Tor, VPNs, DoH/DoT, QUIC).

## Architecture

See [`diagrams/architecture.png`](diagrams/architecture.png) for the full block diagram.

Traffic from the client VM is transparently redirected at the gateway (no client-side
configuration) through the following stack:

```
[Client VM] -> [OPNsense Gateway] -> [Internet]
                 |
                 |-- Squid (transparent proxy, SNI-peek HTTPS inspection via internal CA)
                 |-- SquidGuard (domain/URL blacklisting)
                 |-- Unbound (DNS resolver, port 53 NAT hijack, DNS lockdown)
                 |-- Firewall rules (block QUIC/UDP-443, DoT/853, DoH provider IPs,
                 |                   Tor OR port 9001, VPN protocol ports)
                 |-- Suricata (IDS/IPS — custom rules for Tor/VPN signatures,
                                no-SNI TLS handshakes)
```

## Stack

| Component      | Role                                          | License   |
|-----------------|-----------------------------------------------|-----------|
| OPNsense        | Core filtering gateway                         | BSD 2-Clause |
| Squid           | Transparent proxy, SNI-peek HTTPS inspection    | GPL-2.0   |
| SquidGuard      | Domain/URL blacklisting                         | GPL-2.0   |
| Unbound         | DNS resolver, DNS-layer lockdown                | BSD-3-Clause |
| Suricata        | Intrusion detection / prevention                | GPL-2.0   |
| pfBlockerNG     | DNS-layer blocklist management                  | GPL-2.0   |

Full SBOM: [`docs/SBOM.md`](docs/SBOM.md)

## What this gateway blocks / detects

- Standard HTTP/HTTPS traffic to blacklisted domains (Squid + SquidGuard)
- DNS queries for blocked domains (Unbound + pfBlockerNG)
- DNS-over-HTTPS (DoH) to known public resolver IPs
- DNS-over-TLS (DoT, port 853)
- QUIC / HTTP-3 (UDP 443) — forces fallback to inspectable TCP HTTPS
- Tor (OR port 9001, exit/guard node IP aliases, no-SNI TLS handshake signature)
- VPN protocols — WireGuard (UDP 51820), OpenVPN (1194), IKEv2/IPsec (500, 4500)

## Known gaps

Documented honestly in [`docs/shortcomings.md`](docs/shortcomings.md), including the
Firefox built-in VPN's ability to fall back to port 443 (bypassing the WireGuard-port
block without deep packet inspection), and other tested bypass results.

## Repo layout

```
configs/
  squid/       Squid transparent proxy + SNI-peek config, SquidGuard blacklist
  suricata/    Custom IDS rules (Tor/VPN detection)
  firewall/    OPNsense firewall rule exports (block rules, aliases)
  dns/         Unbound config, DNS lockdown rules
docs/
  SBOM.md
  shortcomings.md
  NETS1037_Project2_Writeup.docx
diagrams/
  architecture.png
```

## Disclaimer

Built for coursework (NETS1037) in an isolated lab environment (VirtualBox VMs).
Not intended for production use as-is — see shortcomings doc for known limitations.
