# Current State vs. Planned Direction

## Current state

The cluster is online and running core services. The focus so far has been on getting the platform stable, backups validated, and essential services running in isolated environments before moving on to networking and hardening work.

**Platform**
- Proxmox VE cluster across `proxmox-01`–`proxmox-04`

**Services**
- Pi-hole — dual instances for redundant DNS filtering
- Uptime Kuma — availability monitoring
- Dashy — service dashboard
- Home Assistant — running in a dedicated VM
- Media stack — isolated in a dedicated VM with bulk disk passthrough

**Backups**
- Proxmox Backup Server (PBS) handling weekly backups, retention, garbage collection, and integrity verification

---

## Planned direction

The next phases shift focus toward network security and operational hardening — the infrastructure foundation is in place, and the gap between the current flat network and a properly segmented environment is the priority.

**Networking and edge**
- Replace ISP router as edge device with a dedicated OPNsense deployment on dual-NIC hardware
- Implement VLAN segmentation via managed switch with defined firewall policy

**Operational maturity**
- Restore testing runbook and periodic restore drills
- Hardening checklist and review
- Alerting refinements
