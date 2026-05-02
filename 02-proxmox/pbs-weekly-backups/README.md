# PBS Weekly Backups (Proxmox to PBS)

## Summary

This section documents the homelab backup posture using Proxmox Backup Server (PBS): weekly Proxmox-initiated backups, centralized retention, integrity verification, and garbage collection.

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

This keeps the backup architecture clear. Proxmox is responsible for creating backups, while PBS is responsible for retaining, verifying, and maintaining them.

---

## Pages

- [PBS storage active on all nodes](01-storage-active-on-all-nodes.md)
- [Proxmox backup job configuration](02-proxmox-backup-job.md)
- [Retention strategy, centralized on PBS](03-retention-strategy.md)
- [PBS maintenance jobs, prune / GC / verify](04-pbs-maintenance-jobs.md)

---

## Current Status

PBS is the centralized backup platform for the Proxmox cluster.

The backup design includes:

- Shared PBS storage configured in Proxmox
- Weekly backup job configuration
- Centralized retention on PBS
- Scheduled prune, garbage collection, and verification jobs
- Post-cutover PBS addressing and backup path validation

After the OPNsense VLAN cutover, PBS was migrated onto the Management / Servers VLAN and validated on the new addressing scheme. Proxmox nodes can reach PBS, the PBS storage target remains available in Proxmox, and backup paths have been validated after the network migration.

PBS should continue to be included in periodic backup and restore validation as part of normal operations.
