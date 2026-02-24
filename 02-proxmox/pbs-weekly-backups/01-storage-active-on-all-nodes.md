# PBS Storage — Active on All Nodes

## Goal

Ensure all cluster nodes target the same backup storage consistently, regardless of guest placement.

---

## Current state

| Detail | Value |
|---|---|
| Proxmox storage ID | `pbs-t7` |
| PBS datastore | `t7-backups` |
| PBS server | `192.168.x.x` |
| Active on | `proxmox-01`, `proxmox-02`, `proxmox-03`, `proxmox-04` |

---

## Why it matters

Having a single shared backup target across all nodes means backup policy is consistent by default — a guest can be migrated or placed on any node without changing how or where it gets backed up. It also avoids per-node backup silos, which simplifies retention management and leaves room to add nodes or guests without redesigning the backup architecture.
