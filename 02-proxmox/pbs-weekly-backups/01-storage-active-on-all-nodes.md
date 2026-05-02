# PBS Storage Active on All Nodes

## Goal

Ensure all cluster nodes target the same backup storage consistently, regardless of guest placement.

After the OPNsense VLAN cutover, PBS was migrated onto the Management / Servers VLAN and validated from the Proxmox nodes.

---

## Current State

| Detail | Value |
|---|---|
| Proxmox storage ID | `pbs-t7` |
| PBS datastore | `t7-backups` |
| PBS server | Management / Servers VLAN address |
| Active on | `proxmox-01`, `proxmox-02`, `proxmox-03`, `proxmox-04` |
| Post-cutover status | Online and validated after VLAN migration |

Full IP addresses are intentionally redacted.

---

## Why It Matters

Having a single shared backup target across all nodes means backup policy is consistent by default. A guest can be migrated or placed on any node without changing how or where it gets backed up.

It also avoids per-node backup silos, simplifies retention management, and leaves room to add nodes or guests without redesigning the backup architecture.

---

## Post-Cutover Validation

After the OPNsense VLAN cutover, PBS reachability and storage availability were validated against the new addressing scheme.

Validation confirmed:

- Proxmox nodes can reach PBS on the Management / Servers VLAN
- The PBS storage target remains available in Proxmox
- The expected datastore is still used
- Backup paths survived the network migration

Periodic backup and restore validation remains part of normal operations.
