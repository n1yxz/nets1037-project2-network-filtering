# Known Shortcomings & Bypass Results

Documented honestly per project requirements — these are real gaps found during testing,
not hypothetical concerns.

## 1. Firefox built-in VPN (port fallback)

Firefox's built-in VPN (version 149+) adapted from its expected port (9001) to port 443
when the OR port was blocked, bypassing our block without triggering deep packet
inspection. Since port 443 carries normal HTTPS traffic, our current ruleset cannot
distinguish this from legitimate encrypted browsing without full DPI.

**Evidence gap:** This bypass also produced **zero log evidence** — no Squid access log
entry, no fired firewall rule, no Suricata alert. This is the most significant blind spot
found during testing.

**Mitigation path (not implemented):** TLS fingerprinting (JA3/JA3S) at the Suricata layer
could distinguish VPN-wrapped TLS from standard browser TLS, but this requires additional
rule tuning beyond project scope.

## 2. QUIC / HTTP-3

QUIC (UDP/443) silently bypasses TCP-only interception since Squid only inspects TCP
traffic. Mitigated by explicitly blocking UDP/443 at the firewall, forcing browsers to
fall back to TCP-based HTTPS where inspection applies — but this is a block, not true
inspection of QUIC itself.

## 3. DNS-over-HTTPS (DoH) to unknown resolvers

We block DoH to known public resolver IPs (Cloudflare, Google, etc.) via IP-list aliases,
but any DoH resolver not on our list is not blocked. This is an IP-blacklist approach,
not protocol-level detection — a private or self-hosted DoH resolver would bypass it
entirely.

## 4. Blacklist maintenance burden

Domain/IP blacklists (SquidGuard, pfBlockerNG, Tor exit/guard node lists) are static
snapshots. In a real deployment, these require continuous updates; a stale list degrades
filtering effectiveness over time without a scheduled update mechanism.

## 5. GPG signature warning (Flathub package)

During Mozilla VPN installation via Flathub, apt/Flatpak surfaced a signature warning:
`GPG verification enabled, but no signatures found (use gpg-verify=false in remote config
to disable)`. The install completed and the package verified as coming from the correct
origin (`org.flathub.Stable`) via `flatpak remote-info`, but the fact that Flatpak allows
an unsigned pull through with only a warning (not a hard failure) is a notable
supply-chain gap — worth flagging even though it wasn't a filtering-gateway component
itself.

## Summary table

| Bypass technique          | Blocked? | Detected/Logged? | Notes |
|-----------------------------|:---:|:---:|-------|
| Standard HTTP/HTTPS to blacklisted domain | Yes | Yes | Squid + SquidGuard |
| DNS query for blocked domain | Yes | Yes | Unbound + pfBlockerNG |
| Tor (default port 9001)     | Yes | Yes | Firewall + Suricata |
| Firefox built-in VPN (port 443 fallback) | No | **No** | Documented gap — needs DPI/JA3 |
| QUIC/HTTP-3                 | Yes (forced fallback) | Partial | UDP/443 blocked outright |
| DoH to known providers      | Yes | Yes | IP-alias based |
| DoH to unlisted providers    | No | No | Not covered by current ruleset |
