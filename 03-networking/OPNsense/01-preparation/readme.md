# OPNsense VLAN Segmentation (Staged in Proxmox VM)

## Summary

This section documents the design and staging of VLAN segmentation and baseline firewall policy in OPNsense before managed switching hardware is in place. The work is being done in a Proxmox VM so the eventual cutover is a controlled transition — not a live rebuild under pressure.

This is written to demonstrate design intent and operational discipline, not as a deployment tutorial.

---

## Goals

- Define VLAN interfaces and IP plan (`10.40.x.0/24` per VLAN)
- Stage DHCP scopes (Kea) disabled until cutover
- Centralize DNS via Pi-hole and NTP via OPNsense across all VLANs
- Enforce default-deny between VLANs with explicit, documented exceptions

---

## Pages

- [Environment and staging approach](01-environment-and-staging.md)
- [VLANs and IP plan](02-vlans-and-ip-plan.md)
- [DNS, NTP, and DHCP posture](03-dns-ntp-dhcp.md)
- [Firewall aliases (maintainability)](04-firewall-aliases.md)
- [Firewall policy by VLAN](05-firewall-policy-by-vlan.md)
- [Staging notes and cutover plan](06-staging-notes-and-cutover.md)
