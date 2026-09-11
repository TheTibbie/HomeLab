# PBS Weekly Backups (Proxmox to PBS)

## Summary

This section documents the homelab backup posture using Proxmox Backup Server (PBS): staggered weekly Proxmox backups, centralized retention, integrity verification, and garbage collection.

The goal is a backup pipeline that is scheduled, validated, and maintained, not just configured once and forgotten.

---

## Backup Model

Proxmox handles the backup job scheduling and transfers guest backups to PBS.

PBS handles:

- Centralized retention
- Pruning
- Garbage collection
- Backup verification
- Datastore maintenance

The original single Sunday backup job was later replaced with staggered per-node jobs across Saturday and Sunday. This reduced concurrent load on the PBS path and gave the `proxmox-04` workload its own backup window.

---

## Current Schedule

| Schedule | Node |
|---|---|
| Saturday `01:00` | `proxmox-01` |
| Saturday `04:00` | `proxmox-02` |
| Sunday `01:00` | `proxmox-03` |
| Sunday `04:00` | `proxmox-04` |

PBS maintenance is kept outside those backup windows:

| Job | Schedule |
|---|---|
| Verify | Monday `01:00` |
| Prune | Wednesday `02:00` |
| Garbage Collection | Wednesday `03:00` |

---

## Pages

- [PBS storage active on all nodes](01-storage-active-on-all-nodes.md)
- [Proxmox backup job configuration](02-proxmox-backup-job.md)
- [Retention strategy, centralized on PBS](03-retention-strategy.md)
- [PBS maintenance jobs, prune / GC / verify](04-pbs-maintenance-jobs.md)

---

## Current Status

PBS remains the centralized backup platform for the Proxmox cluster.

The current design includes:

- Shared PBS storage configured across all Proxmox nodes
- Staggered per-node guest backup jobs
- Snapshot-based backups
- Centralized retention on PBS
- Scheduled prune, garbage collection, and verification jobs
- Manual backup validation during the most recent backup review
- CT 110 added to the `proxmox-03` backup schedule after the internal DNS / reverse proxy project

Periodic backup and restore validation remains part of normal operations.
