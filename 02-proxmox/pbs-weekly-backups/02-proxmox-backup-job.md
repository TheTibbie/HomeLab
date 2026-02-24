# Proxmox Backup Job (Weekly)

## Goal

Run a predictable, low-touch weekly backup to PBS using snapshot mode, covering all production guests.

---

## Current configuration

| Detail | Value |
|---|---|
| Schedule | Sundays at `01:00` |
| Mode | `Snapshot` |
| Compression | `ZSTD (fast and good)` |
| Selection mode | `Include selected VMs` |
| Target | `pbs-t7` |
| Notes template | `{{guestname}}` |

**Guests in scope**

| ID | Guest | Type | Node |
|---|---|---|---|
| VM 100 | `media-vm` | Virtual Machine | `proxmox-01` |
| CT 101 | `pihole` | LXC Container | `proxmox-02` |
| CT 102 | `pihole-b` | LXC Container | `proxmox-03` |
| CT 103 | `uptime-kuma` | LXC Container | `proxmox-02` |
| VM 104 | `opnsense-lab` | Virtual Machine | `proxmox-04` |
| VM 105 | `homeassistant` | Virtual Machine | `proxmox-02` |
| CT 120 | `dashy` | LXC Container | `proxmox-01` |

---

## Why snapshot mode

Snapshot mode allows backups to run against live guests without shutting them down, keeping services continuously available. For a homelab where most workloads run 24/7, it's the right default — suspend or stop mode would introduce unnecessary downtime on a weekly basis.
