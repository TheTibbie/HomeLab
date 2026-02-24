# Retention Strategy: Centralized on PBS

## Decision

Retention is managed centrally on PBS rather than via the Proxmox backup job. This keeps retention logic in one place and avoids a permission-related failure state that surfaced during initial setup.

---

## Background

During initial configuration, backup jobs were reporting failure despite data transferring to PBS successfully. The root cause was post-backup prune behavior being triggered from the Proxmox side — the backup upload itself worked, but the prune step was failing on permissions, which caused the overall job to report as failed.

The fix was straightforward: disable pruning from the Proxmox job and let PBS own retention entirely.

---

## Current approach

- Proxmox backup jobs handle transfer only — no pruning configured on the Proxmox side
- Retention, pruning, and GC are handled by scheduled PBS maintenance jobs (documented in [PBS maintenance jobs](04-pbs-maintenance-jobs.md))
<img width="1668" height="491" alt="image" src="https://github.com/user-attachments/assets/03802301-8673-4515-aab8-0b492cdb548c" />
<img width="1728" height="274" alt="image" src="https://github.com/user-attachments/assets/ae907fc4-5b37-41a3-b5ee-2c56e583db84" />
<img width="1302" height="265" alt="image" src="https://github.com/user-attachments/assets/e8789c42-98a5-415e-beea-acee2fb88cff" />

