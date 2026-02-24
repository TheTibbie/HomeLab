# Inventory and Deployment Notes

## Purpose

Provide DNS filtering and domain blocking at the network layer using lightweight Proxmox LXC containers, with enough redundancy to survive a single node failure or maintenance window.

---

## Inventory

| Detail | `pihole` | `pihole-b` |
|---|---|---|
| CT ID | 101 | 102 |
| Node | `proxmox-02` | `proxmox-03` |
| LAN IP | `192.168.x.x` | `192.168.x.x` |

---

## Current configuration

- DHCP is handled by the ISP router
- LAN clients receive both Pi-hole instances as DNS servers via router DHCP
- Upstream DNS: Google (both instances)

---

## Why two instances

Running a second instance on a separate node means DNS filtering stays active during maintenance on either node, and a node failure doesn't take the whole filtering layer down. It's a low-cost redundancy measure given that the containers are small and the nodes are already in the cluster.
