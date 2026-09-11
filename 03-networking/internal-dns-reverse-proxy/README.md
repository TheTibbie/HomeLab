# Internal DNS and Reverse Proxy

## Summary

This section documents the internal `home.arpa` naming scheme and Caddy reverse proxy used for stable, human-friendly access to homelab services.

The project was designed for internal use only. It does not depend on public DNS and does not expose the reverse proxy to the WAN.

---

## Design

The general application path is:

```text
Client
  ↓
Pi-hole DNS
  ↓
Caddy reverse proxy
  ↓
Internal service
```

Infrastructure management interfaces use internal DNS names directly and are not unnecessarily placed behind Caddy.

---

## Current Model

The current design includes:

- `home.arpa` internal namespace
- Matching local DNS records on both Pi-hole instances
- Caddy running in a dedicated unprivileged LXC
- Port-free HTTP URLs for selected internal applications
- DNS-only names for Proxmox, PBS, OPNsense, Pi-hole, and Omada management
- No WAN port forwards for the reverse proxy
- Existing direct IP:port access retained as a fallback
- Reverse-proxy health integrated into the monitoring stack

---

## Pages

- [Internal DNS](01-internal-dns.md)
- [Reverse proxy](02-reverse-proxy.md)
- [Issues encountered and fixes](03-issues-and-fixes.md)

---

## Current Status

The internal DNS and reverse proxy implementation is complete and operational.

The design intentionally keeps infrastructure management interfaces simple. DNS provides readable names for those systems, while Caddy is used only where removing application port numbers provides a practical day-to-day benefit.
