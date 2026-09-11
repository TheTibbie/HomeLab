# Retention Strategy: Centralized on PBS

## Decision

Retention is managed centrally on PBS rather than through the Proxmox backup jobs.

This keeps retention logic in one place and avoids a permission-related failure state that surfaced during initial setup.

---

## Background

During initial configuration, backup jobs were reporting failure despite data transferring to PBS successfully.

The root cause was post-backup prune behavior being triggered from the Proxmox side. The backup upload itself worked, but the prune step was failing on permissions, which caused the overall job to report as failed.

The fix was straightforward: disable pruning from the Proxmox jobs and let PBS own retention entirely.

---

## Current Approach

- Proxmox backup jobs handle transfer only
- No pruning is configured on the Proxmox backup jobs
- Retention, pruning, garbage collection, and verification are handled by scheduled PBS maintenance jobs
- Backup jobs are staggered across Saturday and Sunday
- PBS maintenance is intentionally scheduled outside the guest backup windows

Current prune retention:

```text
keep-last=4
keep-monthly=3
```

This keeps the most recent restore points available while also retaining a small amount of longer-term monthly history.

Maintenance jobs are documented in [PBS maintenance jobs](04-pbs-maintenance-jobs.md).

---

## Why Centralized Retention Still Makes Sense

The newer staggered backup design changed when guest backups run, but it did not change where retention belongs.

Keeping retention on PBS means all four Proxmox jobs follow the same lifecycle policy without duplicating prune settings across the cluster.

It also keeps backup creation and backup lifecycle management separate:

- Proxmox creates and transfers the backups
- PBS decides what is retained
- PBS reclaims unused space
- PBS verifies retained backup data

---

## Current Status

PBS remains the centralized retention point for the Proxmox cluster.

The newer schedule has also moved prune, garbage collection, and verification away from the weekend guest backup windows to reduce overlap between backup traffic and maintenance activity.

---

<img width="1668" height="491" alt="PBS retention configuration" src="https://github.com/user-attachments/assets/03802301-8673-4515-aab8-0b492cdb548c" />

<img width="1728" height="274" alt="PBS prune job configuration" src="https://github.com/user-attachments/assets/ae907fc4-5b37-41a3-b5ee-2c56e583db84" />

<img width="1302" height="265" alt="PBS retention validation screenshot" src="https://github.com/user-attachments/assets/e8789c42-98a5-415e-beea-acee2fb88cff" />
