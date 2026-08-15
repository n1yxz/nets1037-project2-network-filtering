# Test Results — What Worked, What Didn't

Consolidated results from actual build/test sessions. Each row reflects a real test
performed during the project, not a theoretical/expected outcome.

## Core filtering — worked

| # | Test | Mechanism | Result | Evidence |
|---|------|-----------|--------|----------|
| 1 | Blocked domain via HTTP/HTTPS | Squid transparent proxy + SquidGuard blacklist | **Blocked** | Squid access.log shows denied request; browser shows block page |
| 2 | DNS query for blocked domain | Unbound + pfBlockerNG, port 53 NAT hijack | **Blocked** | `resolvectl status` confirms client locked to gateway DNS; query sinkholed |
| 3 | Bare apex domain block (e.g. `example.com` with no `www`) | Squid url_regex blacklist | **Fixed after bug found** | Initial test failed to match bare apex — required adding both dotted (`.example.com`) and non-dotted entries; retest confirmed match |
| 4 | Tor connection (default OR port 9001) | Firewall block on port 9001 + IP alias for known Tor exit/guard nodes | **Blocked** | Firewall rule hit counter incremented; connection failed at handshake |
| 5 | Tor traffic pattern (no-SNI TLS handshake) | Custom Suricata rule alerting on TLS with no SNI extension | **Detected/logged** | Suricata alert fired in `eve.json` / alerts UI during Tor connection attempt |
| 6 | QUIC/HTTP-3 (UDP/443) | Firewall block on UDP/443 | **Blocked (forced fallback)** | Browser fell back to TCP HTTPS, which was then inspectable by Squid |

## Bugs found and fixed during testing

| Bug | Symptom | Fix |
|-----|---------|-----|
| NAT redirect rules not auto-created | Transparent proxy plugin implied automatic NAT setup, but traffic wasn't being redirected | Manually added NAT redirect rules for ports 80/443 to Squid |
| Squid segfault | SSL-bump crashed on startup | Was using a plain leaf certificate instead of a CA certificate — reissued as proper internal CA |
| Squid blacklist not matching bare domains | Blocked domain still loaded without `www.` | Added both dotted and non-dotted entries per domain |
| Block appeared inactive | Site loaded despite block rule in place | Browser cache was serving a previously-cached copy — cleared cache / used private window to confirm block was actually working |
| QUIC bypassing inspection | Blocked sites still loaded over HTTP/3 | Added explicit UDP/443 block; TCP-only interception wasn't catching QUIC |
| Firewall block rules silently ignored | Rules configured correctly but had no effect | Rules were positioned below the default allow rule — pf/OPNsense evaluates first-match, so reordered blocks above the allow rule |

## Bypass testing — confirmed bypass (real gap, not hypothetical)

| # | Test | Mechanism | Result | Evidence |
|---|------|-----------|--------|----------|
| 7 | Firefox built-in VPN (v149+) | Expected to hit port-9001 block; VPN instead adapted to port 443 | **Bypassed — not blocked** | Site loaded successfully over the VPN despite firewall/Suricata rules in place |
| 8 | Same Firefox VPN bypass — logging check | Checked Squid access.log, firewall rule counters, Suricata alerts during the bypass | **Zero evidence generated** | No Squid log entry, no firewall rule fired, no Suricata alert — confirmed genuine blind spot |

This is the project's most significant honest finding: a real, reproducible technique that
defeats the filtering stack **and leaves no trace**, which is a stronger and more useful
result for a security project than a clean 100%-block outcome would be.

## Summary

| Category | Outcome |
|----------|---------|
| Domain/URL blacklisting | Working, verified |
| DNS-layer blocking | Working, verified |
| Tor (default configuration) | Blocked + detected, verified |
| QUIC/HTTP-3 | Blocked (forced fallback), verified |
| Firefox built-in VPN (port-443 fallback) | **Confirmed bypass, no log evidence** |
| Native VPN clients (Mozilla VPN, Proton VPN, etc.) | Not tested — lab environment decommissioned before this test could run |

## Note on scope

The native VPN client test (installing Mozilla VPN or Proton VPN directly and testing
against the WireGuard/OpenVPN/IKEv2 port blocks) was planned but not completed — the lab
VMs were decommissioned before this specific test could be run. The Firefox built-in VPN
result above stands as the project's actual, tested VPN-bypass evidence.
