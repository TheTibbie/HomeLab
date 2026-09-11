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
| Backup model | Staggered per-node jobs |
| Post-cutover status | Online and validated after VLAN migration |

Full IP addresses are intentionally redacted.

---

## Current Backup Layout

The original single weekly backup job was replaced with staggered jobs so the nodes do not all push backups to PBS at the same time.

| Node | Backup Window |
|---|---|
| `proxmox-01` | Saturday at `01:00` |
| `proxmox-02` | Saturday at `04:00` |
| `proxmox-03` | Sunday at `01:00` |
| `proxmox-04` | Sunday at `04:00` |

All four jobs still use the same `pbs-t7` storage target.

The staggered design was introduced after backup reliability testing showed that the PBS path performed better when larger guests were not competing with the rest of the cluster at the same time.

---

## Why It Matters

Having a single shared backup target across all nodes means backup policy is consistent by default. A guest can be migrated or placed on another node without changing the PBS storage design.

Separating the backup jobs by node also reduces simultaneous load on the PBS path while keeping the storage and retention model centralized.

---

## Validation

Validation confirmed:

- All four Proxmox nodes can reach PBS
- The `pbs-t7` storage target is active across the cluster
- The expected datastore is still used
- Scheduled jobs are distributed across separate backup windows
- Manual backup testing has completed successfully against the current PBS target

Periodic backup and restore validation remains part of normal operations.
