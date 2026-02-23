# Public Repo Redaction Policy

## Goal

Share architecture and operational work publicly without exposing sensitive details or creating avoidable attack surface.

---

## What gets redacted

**Network identifiers**
- LAN IPs written as `192.168.x.x`
- WAN IPs, public-facing addresses, and open port listings omitted

**Credentials and secrets**
- Tokens, API keys, join secrets, and credentials of any kind
- VPN configs, WireGuard/OpenVPN pre-shared keys
- PBS fingerprints and datastore paths with identifying information

**Device and identity details**
- Usernames that reveal naming patterns
- Full client/device inventories, MAC addresses, serial numbers

---

## Screenshot hygiene

Screenshots are cropped to show only what supports the point being documented. IPs, usernames, hostnames with identifying information, and any external exposure details are blurred before publishing. Where a screenshot would require heavy redaction, a text summary is used instead.

---

## Security posture

This environment is built to minimize exposure. Remote access runs through Tailscale rather than direct port forwarding. VLAN segmentation and firewall policy are scoped as an explicit next phase — the current flat network is a known interim state, not an oversight.
