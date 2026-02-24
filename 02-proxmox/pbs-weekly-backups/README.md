# PBS Weekly Backups (Proxmox → PBS)

## Summary

This section documents the homelab backup posture using Proxmox Backup Server (PBS): weekly Proxmox-initiated backups, centralized retention, integrity verification, and garbage collection. The goal is a backup pipeline that's scheduled, validated, and maintained — not just configured and forgotten.

---

## Pages

- [PBS storage active on all nodes](01-storage-active-on-all-nodes.md)
- [Proxmox backup job configuration](02-proxmox-backup-job.md)
- [Retention strategy (centralized on PBS)](03-retention-strategy.md)
- [PBS maintenance jobs (prune / GC / verify)](04-pbs-maintenance-jobs.md)
- [Notifications — intentionally skipped](05-notifications-skipped.md)
