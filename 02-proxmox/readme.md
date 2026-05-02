# Proxmox Platform

## Summary

This section documents the Proxmox VE platform that powers the homelab: cluster posture, node profiles, guest placement, storage strategy, and backup operations. The goal is to demonstrate platform ownership and operational thinking — availability, recoverability, and clarity — not to provide a step-by-step deployment tutorial.

---

## What's in this section

- Cluster membership, transport, and quorum posture
- Node hardware profiles and capacity context
- Guest inventory (VMs and CTs) with placement rationale
- Storage definitions: local LVM, shared ISO via NFS, PBS backup storage
- Backup posture (Proxmox → PBS) including retention, GC, and integrity verification

---

## Structure

| Section | Contents |
|---|---|
| `cluster/` | Cluster state, quorum, and standards |
| `nodes/` | Node hardware profiles |
| `guests/` | VM and CT inventory — what runs where |
| `storage/` | Storage design — ISO library, media disk passthrough |
| `pbs-weekly-backups/` | Backup jobs and PBS maintenance posture |
| `uptime-kuma/` | Monitoring service |
| `dashy/` | Service dashboard |

---

## Redaction standard

This section follows the repo-wide redaction policy documented in [Public repo redaction policy](../01-overview/03-redaction-policy.md). In brief:

- LAN IPs written as `192.168.x.x`
- Not published: tokens, join secrets, VPN configs, PBS fingerprints, or client inventories
- Screenshots cropped and blurred if they show IPs, WAN details, usernames, or exposed ports

---

![Proxmox cluster overview showing four-node membership](https://github.com/user-attachments/assets/50436868-09be-49c8-8079-79538e1bd68a)

