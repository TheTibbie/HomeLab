# Proxmox Cluster (Homelab)

## Summary

This section documents the Proxmox VE cluster named `Homelab`, including platform version, membership, quorum state, and networking baseline. It covers current posture and operational notes, not a deployment walkthrough.

After the OPNsense VLAN cutover, Proxmox management moved from the old flat LAN to the Management / Servers VLAN. Corosync was also updated to use the new management placement after quorum was temporarily lost during the initial cutover.

---

## Platform

| Detail | Value |
|---|---|
| Proxmox VE | `9.1.1` (kernel `6.17.x`) |
| Cluster name | `Homelab` |
| Transport | `knet` |
| Secure auth | `on` |
| Quorum | Healthy, 4 nodes, 4 expected votes, quorum 3 |

---

## Networking Baseline

The Proxmox cluster now operates on the Management / Servers VLAN.

Current baseline:

- Proxmox management is on the Management / Servers VLAN
- OPNsense is the active router/firewall for the environment
- DHCP is handled by OPNsense
- Each node uses a Linux bridge for VM/CT connectivity
- Corosync cluster traffic uses updated management VLAN ring addresses

The previous flat LAN was the original pre-cutover state. It is retained in older preparation notes as historical context, but it is no longer the active network design.

---

## Cutover Note

During the OPNsense VLAN cutover, the cluster temporarily lost quorum because Corosync was still referencing the old flat-network addresses. The Corosync ring addresses were updated to the new Management / Servers VLAN addresses, the configuration version was bumped, and the cluster re-formed successfully.

The cutover and recovery process is documented in:

`../../03-networking/OPNsense/02-cutover-and-implementation/`

---

## Pages

- [Current membership + quorum notes](01-current-nodes-and-quorum.md)
- [Standards + next steps](02-standards-and-next-steps.md)
