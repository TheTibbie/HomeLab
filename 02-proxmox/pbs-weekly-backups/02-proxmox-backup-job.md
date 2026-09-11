# Proxmox Backup Jobs (Staggered Weekly)

## Goal

Run predictable, low-touch backups to PBS using snapshot mode while avoiding unnecessary concurrent load on the backup path.

The original design used one Sunday backup job for most guests. After reliability testing, the schedule was split into four per-node jobs across Saturday and Sunday.

---

## Current Configuration

| Detail | Value |
|---|---|
| Mode | `Snapshot` |
| Compression | `ZSTD (fast and good)` |
| Selection mode | `Include selected VMs` |
| Target | `pbs-t7` |
| Notes template | `{{guestname}}` |
| Retention | Handled by PBS |

---

## Current Schedule

| Schedule | Node | Guests |
|---|---|---|
| Saturday at `01:00` | `proxmox-01` | 100, 120 |
| Saturday at `04:00` | `proxmox-02` | 101, 103, 105, 200 |
| Sunday at `01:00` | `proxmox-03` | 102, 109, 110 |
| Sunday at `04:00` | `proxmox-04` | 104 |

The Sunday `proxmox-03` job was later expanded to include CT 110 after the internal DNS / reverse proxy project was completed.

---

## Why the Schedule Changed

The previous single-job design created unnecessary competition for the PBS connection.

During testing, VM 104 repeatedly failed scheduled backups while a manual standalone backup completed successfully. PBS was also found to be negotiating at `100 Mb/s`.

Rather than redesigning the network during the backup project, the jobs were staggered by node.

The current design:

- Reduces simultaneous backup traffic
- Gives VM 104 its own backup window
- Adds VM 109 and CT 200, which were missing from the older schedule
- Includes CT 110 after the reverse proxy deployment
- Keeps the same centralized PBS target

---

## Why Snapshot Mode

Snapshot mode allows backups to run against live guests without shutting them down, keeping services continuously available.

For a homelab where most workloads run 24/7, it remains the preferred default. Suspend or stop mode would introduce unnecessary downtime during the weekly backup cycle.

---

## Validation

Manual backup testing confirmed that the PBS path works when jobs are separated from competing backup traffic.

The first full staggered weekend cycle should continue to be reviewed as part of normal backup operations.
