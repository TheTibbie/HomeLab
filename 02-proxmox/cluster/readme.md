# Proxmox Cluster (Homelab)

## Summary

This section documents the Proxmox VE cluster named `Homelab` — platform version, membership, quorum state, and networking baseline. It covers current posture and what's planned next, not a deployment walkthrough.

---

## Platform

| Detail | Value |
|---|---|
| Proxmox VE | `9.1.1` (kernel `6.17.x`) |
| Cluster name | `Homelab` |
| Transport | `knet` |
| Secure auth | `on` |
| Quorum | Healthy — 4 nodes, 4 expected votes, quorum 3 |

---

## Networking baseline

- Flat LAN: `192.168.x.x/24` — ISP router provides gateway and DHCP
- Corosync cluster traffic runs on `ring0` over the main LAN interface
- Each node uses a single bridge: `vmbr0`

VLAN segmentation and a dedicated OPNsense edge are planned — the flat network is a known interim state documented in the [overview](../01-overview/current-vs-planned.md).

---

## Pages

- [Current membership + quorum notes](01-current-nodes-and-quorum.md)
- [Standards + next steps](02-standards-and-next-steps.md)
