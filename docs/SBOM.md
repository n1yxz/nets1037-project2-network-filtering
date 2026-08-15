# Software Bill of Materials (SBOM) — NETS1037 Project 2

| # | Component     | Version (as tested) | License      | Source / Repo                                      | Role in project |
|---|----------------|----------------------|--------------|------------------------------------------------------|------------------|
| 1 | OPNsense       | Version not recorded - lab environment decommissioned before this was captured | BSD 2-Clause | https://github.com/opnsense/core        | Core filtering gateway OS |
| 2 | Squid          | Version not recorded - lab environment decommissioned before this was captured) | GPL-2.0      | https://github.com/squid-cache/squid    | Transparent proxy, SNI-peek HTTPS inspection |
| 3 | SquidGuard     | Version not recorded - lab environment decommissioned before this was captured             | GPL-2.0      | https://github.com/opnsense/plugins (os-squidguard) | Domain/URL blacklisting |
| 4 | Unbound        | Version not recorded - lab environment decommissioned before this was captured         | BSD-3-Clause | https://github.com/NLnetLabs/unbound    | DNS resolver, DNS lockdown |
| 5 | pfBlockerNG    | Version not recorded - lab environment decommissioned before this was captured             | GPL-2.0      | https://github.com/opnsense/plugins (os-pfBlockerNG) | DNS-layer blocklists |
| 6 | Suricata       | Version not recorded - lab environment decommissioned before this was captured | GPL-2.0  | https://github.com/OISF/suricata        | IDS/IPS, Tor/VPN signature detection |
| 7 | Mozilla VPN (client, bypass-test tool) | 2.39.0 | MPL-2.0 | https://github.com/mozilla-mobile/mozilla-vpn-client | Bypass/circumvention testing on client VM |

## Notes

- All components are open-source and permissively/copyleft licensed (no proprietary
  or closed-source dependencies), satisfying the project's licensing requirement.
- Exact OPNsense/Squid/Unbound/Suricata version numbers should be filled in from the
  live system (System > Firmware > Status on OPNsense shows the base + plugin versions).
- Mozilla VPN was installed via Flathub (`org.mozilla.vpn`, commit
  `f5352db8adfda71e8672222e1ecf7748d9a0dea627edaa9af5a00b636ecebee5`) for bypass testing
  on the client VM — not part of the filtering stack itself.
