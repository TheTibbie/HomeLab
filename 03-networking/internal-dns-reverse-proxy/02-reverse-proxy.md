# Reverse Proxy

## Goal

Provide cleaner internal application URLs without exposing services publicly or changing the existing infrastructure management model.

Caddy is used as the internal reverse proxy.

---

## Reverse Proxy Host

The reverse proxy runs in a dedicated Debian LXC on Proxmox.

| Detail | Value |
|---|---|
| Hostname | `reverse-proxy` |
| Type | Unprivileged LXC |
| Reverse proxy | Caddy |
| Configuration | `/etc/caddy/Caddyfile` |
| Start at boot | Enabled |
| Public exposure | None |

The container uses a small resource footprint because its role is limited to internal HTTP proxying.

---

## Access Model

Selected application names resolve to the reverse proxy through Pi-hole.

Caddy then forwards each hostname to the correct internal backend.

Examples documented here include:

```text
http://grafana.home.arpa
http://dashy.home.arpa
http://uptime.home.arpa
http://homeassistant.home.arpa
```

Infrastructure management interfaces such as Proxmox, PBS, OPNsense, Pi-hole, and Omada remain DNS-only and are accessed on their normal management ports.

---

## Firewall Policy

The reverse proxy is reachable only from the internal VLANs that need it.

OPNsense uses a reverse-proxy alias and explicit TCP/80 allow rules where required.

No TCP/443 reverse-proxy rule was added during this phase because the project uses HTTP internally.

No WAN port forward was created.

---

## Direct Access Remains Available

The project did not intentionally remove direct backend access.

That provides a useful fallback during troubleshooting and also keeps the reverse proxy from becoming the only way to administer or validate a service.

The intended troubleshooting model is:

```text
Friendly hostname fails
        ↓
Test backend directly
        ↓
Separate proxy/DNS issue from application issue
```

---

## Monitoring

The reverse proxy has a lightweight health endpoint under the internal namespace.

Blackbox Exporter checks that endpoint so the monitoring system validates the combined DNS + Caddy path.

Existing direct backend probes remain in place. This makes it possible to distinguish a reverse-proxy-path problem from a backend service problem.

---

## Backup

The reverse proxy is covered in two ways:

- The LXC is included in the `proxmox-03` PBS schedule
- The active Caddy configuration is also copied off-host with the configuration backup set

This provides both full guest recovery and a smaller configuration-level recovery option.

---

## Validation

Validation includes:

- Caddy service is active and enabled
- Caddy configuration validates successfully
- Internal DNS resolves the expected hostnames
- Proxied URLs return the expected application response
- Direct backend access still works where intentionally retained
- Blackbox reverse-proxy health returns success
