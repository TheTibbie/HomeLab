# Inventory and Deployment Notes

## Purpose

Provide DNS filtering and domain blocking at the network layer using lightweight Proxmox LXC containers, with enough redundancy to survive a single node failure or maintenance window.

After the OPNsense VLAN cutover, both Pi-hole instances are placed on the Management / Servers VLAN and distributed to clients through OPNsense DHCP.

---

## Inventory

| Detail | `pihole` | `pihole-b` |
|---|---|---|
| CT ID | 101 | 102 |
| Node | `proxmox-02` | `proxmox-03` |
| Network Placement | Management / Servers VLAN | Management / Servers VLAN |

Full IP addresses are intentionally redacted.

---

## Current Configuration

- DHCP is handled by OPNsense
- OPNsense distributes both Pi-hole instances as DNS servers to client VLANs
- Pi-hole provides DNS filtering for client VLANs
- Upstream DNS: Google on both instances
- Pi-hole hosts are allowed to reach upstream DNS through firewall policy
- Direct client DNS bypass is restricted where DNS enforcement is enabled

---

## Why Two Instances

Running a second instance on a separate node means DNS filtering stays active during maintenance on either node, and a node failure does not take the whole filtering layer down.

This is a low-cost redundancy measure because the containers are small and the Proxmox nodes are already part of the cluster.

---

## Post-Cutover Note

Before the OPNsense cutover, DHCP was handled by the ISP router and clients received Pi-hole DNS through router DHCP.

That is no longer the active design. DHCP has moved to OPNsense, and Pi-hole now operates as part of the Management / Servers VLAN infrastructure.
