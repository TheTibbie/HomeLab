# Standards and Next Steps

## Naming conventions

- Nodes follow a sequential naming pattern: `proxmox-01`–`proxmox-04`
- Guests use short, descriptive names — service first, role second (e.g., `pihole`, `pihole-b`, `media-vm`)

---

## Redaction standards

This section follows the repo-wide redaction policy documented in [Public repo redaction policy](../../01-overview/03-redaction-policy.md). In brief:

- Internal IPs written as `192.168.x.x`
- Not published: tokens, keys, join secrets, VPN configs, PBS fingerprints, client inventories, or logs with identifying details
- Screenshots cropped and blurred if they show IPs, WAN info, usernames, or exposed ports

---

## Current baseline

- Flat LAN — no VLAN segmentation yet
- Corosync `ring0` runs on the main LAN interface
- Single Linux bridge per node (`vmbr0`)

This is a known interim state. The networking redesign is the next active phase.

---

## Planned improvements

**Edge and segmentation**
- Promote the OPNsense lab VM to a dedicated edge deployment on dual-NIC hardware
- Implement VLAN segmentation via managed switch with defined firewall policy

**Operational maturity**
- Restore drills and a validated restore runbook
- Operational maturity checklist covering restore validation, change notes, and monitoring scope
