# Proxmox Platform

## Summary

This section documents the Proxmox VE platform that powers the homelab, including cluster posture, node profiles, guest placement, storage strategy, and backup operations. The goal is to demonstrate platform ownership and operational thinking: availability, recoverability, and clarity. This is not a step-by-step deployment tutorial.

After the OPNsense VLAN cutover, Proxmox management and several supporting services are placed on the Management / Servers VLAN. Uptime Kuma and Dashy still run as Proxmox guests, but their active network placement is now documented as part of the post-cutover Management / Servers infrastructure.

---

## What's in this section

- Cluster membership, transport, and quorum posture
- Node hardware profiles and capacity context
- Guest inventory with placement rationale
- Storage definitions, including local LVM, shared ISO via NFS, and PBS backup storage
- Backup posture from Proxmox to PBS, including retention, garbage collection, and integrity verification
- Supporting service guests such as Uptime Kuma and Dashy, now operating on the Management / Servers VLAN

---

## Structure

| Section | Contents |
|---|---|
| `cluster/` | Cluster state, quorum, and standards |
| `nodes/` | Node hardware profiles |
| `guests/` | VM and CT inventory, including what runs where |
| `storage/` | Storage design, including ISO library and media disk passthrough |
| `pbs-weekly-backups/` | Backup jobs and PBS maintenance posture |
| `uptime-kuma/` | Monitoring service running as a Proxmox guest on the Management / Servers VLAN |
| `dashy/` | Service dashboard running as a Proxmox guest on the Management / Servers VLAN |

---

## Redaction Standard

This section follows the repo-wide redaction policy documented in [Public repo redaction policy](../01-overview/03-redaction-policy.md).

In brief:

- Internal IPs are redacted or generalized
- Tokens, join secrets, VPN configs, PBS fingerprints, and client inventories are not published
- Screenshots are cropped or blurred if they show IPs, WAN details, usernames, exposed ports, or other sensitive identifiers

---

![Proxmox cluster overview showing four-node membership](https://github.com/user-attachments/assets/50436868-09be-49c8-8079-79538e1bd68a)
