# bind9

Internal DNS for my homelab: BIND 9.20 in Docker on the home VM, serving the
`home.matthomelab.xyz` zone.

> A published snapshot of a repository that runs on my self-hosted GitLab.
> More about the lab: [homelab](https://github.com/MatejK97/homelab)

## Design

- **One wildcard record** points every `*.home` name at the VPS's WireGuard
  address, where Traefik routes it. A new service needs no DNS change.
- **Cloudflare as fallback.** The same wildcard is published in Cloudflare
  (DNS only), so names still resolve when the home VM is off. They resolve to
  a VPN address, so they're only reachable over WireGuard anyway.
- **Recursive resolver for the LAN:** queries for anything else are
  forwarded, recursion is limited to internal networks, and DNSSEC validation
  is on.
- **Persistent working directory:** `directory "/var/cache/bind"` is on a
  mounted volume, so the DNSSEC root trust anchors BIND learns survive
  container recreation. (They didn't once, which is how I learned this.)

## CI

- **Merge requests:** `named-checkconf -z` parses the configuration and loads
  every zone, so a broken record never reaches DNS.
- **Merge to `main`:** the shared pipeline from
  [ci-templates](https://github.com/MatejK97/ci-templates) deploys to the home
  VM over WireGuard (`REMOTE_HOST`) and restarts BIND
  (`DEPLOY_RESTART`), since it only reads zone files at startup.
