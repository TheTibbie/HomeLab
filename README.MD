# Homelab Architecture

## Summary

This repository documents the design and ongoing build-out of a self-hosted homelab environment built around a 4-node Proxmox VE cluster. The focus is on virtualization, network segmentation, and operational discipline — not just getting services running, but running them with the same practices that matter in production: backups, monitoring, change documentation, and a clear path forward. The environment is actively being built and documented as it evolves.

> **Status: Build Phase** — Proxmox cluster, backups, monitoring, DNS filtering, and core services are online. VLAN segmentation and OPNsense edge deployment are next.

---

## Design intent

The lab is built to be deliberately structured rather than convenient. A few key decisions reflect that:

**Proxmox over simple Docker hosts.** Running workloads in VMs and containers under Proxmox provides real isolation, snapshotting, and live migration capability — and it mirrors how infrastructure is actually managed at scale. Each service gets its own boundary rather than sharing a host OS.

**Dedicated backup infrastructure.** Proxmox Backup Server runs as a separate service handling scheduled backups, retention, garbage collection, and integrity verification. The goal is a backup posture that's scheduled and validated, not ad hoc.

**DNS filtering before edge firewalling.** Pi-hole is deployed in a redundant pair (two nodes) ahead of the planned OPNsense rollout. Incremental security improvements were preferred over waiting for the full networking redesign.

**Flat network as a known interim state.** The current flat LAN is intentional — the network segmentation work (VLANs, firewall policy, managed switch) is scoped and planned, not forgotten. The OPNsense lab VM already exists for staging.

**Remote access via Tailscale.** Rather than port-forwarding into the lab, Tailscale handles remote access. This avoids exposing services directly to the internet while the edge architecture is still being built.

---

## Current state

### Platform

- **Proxmox VE** `9.1.1` (kernel `6.17.x`) — 4-node cluster (`Homelab`), transport `knet`, secure auth enabled, quorum healthy (4 nodes, 4 expected votes)

| Node | Hardware | RAM |
|---|---|---|
| `proxmox-01` | Dell OptiPlex, i5-6500 | ~15 GiB |
| `proxmox-02` | ThinkCentre, i5-4570 | ~11 GiB |
| `proxmox-03` | ThinkCentre, i5-4570 | ~11 GiB |
| `proxmox-04` | ThinkCentre, i5-4570 | ~11 GiB |

### Networking

- Flat LAN: `192.168.x.x/24`, gateway/DHCP via ISP router
- Each node: single Linux bridge (`vmbr0`) on `nic0`
- VLAN segmentation: planned, not yet implemented
- Remote access: Tailscale

### Storage

| Name | Type | Notes |
|---|---|---|
| `local` | Directory | `/var/lib/vz`, per-node |
| `local-lvm` | LVM-thin | `data` thinpool in VG `pve`, per-node |
| `iso-nfs` | NFS (shared) | PBS-hosted ISO library, available cluster-wide |
| `pbs-t7` | PBS (shared) | Backup target — datastore `t7-backups` |

Bulk media disk is passed through directly to the media VM (see `02-proxmox/storage/`).

### Workloads

**Platform**
- Proxmox VE cluster across all four nodes
- Proxmox Backup Server (PBS)

**Network services**
- Pi-hole primary (`pihole`, CT 101) — `proxmox-02`
- Pi-hole secondary (`pihole-b`, CT 102) — `proxmox-03`

**Monitoring and visibility**
- Uptime Kuma (CT 103) — `proxmox-02`
- Dashy dashboard (CT 120) — `proxmox-01`

**Smart home**
- Home Assistant (VM 105) — `proxmox-02`

**Media**
- Media VM (VM 100) — `proxmox-01`, bulk media disk passthrough

**Networking lab**
- OPNsense lab VM (VM 104) — `proxmox-04` (staging; ISP router is still active edge/DHCP)

### Backups

Backups run on a weekly Proxmox job to PBS:

- Backup job: Sundays at 01:00, snapshot mode
- PBS retention: keep-last=2
- Weekly maintenance chain: prune (02:00) → GC (03:00) → verify (04:00, re-verify after 30 days)

---

## Roadmap

### Phase 1 — Foundation *(in progress)*
- [x] Proxmox cluster online
- [x] Shared ISO library via NFS
- [x] Weekly PBS backups + maintenance pipeline
- [x] Core services online (Pi-hole, Uptime Kuma, Dashy, Home Assistant)
- [ ] Restore testing runbook + periodic restore drills

### Phase 2 — Services *(in progress)*
- [x] Media stack deployed in isolated VM
- [ ] Service documentation expanded as the environment evolves

### Phase 3 — Networking & Security *(planned)*
- [ ] OPNsense edge deployment on dedicated dual-NIC hardware
- [ ] VLAN segmentation + firewall policy
- [ ] Policy decisions documented with reasoning

### Phase 4 — Operational Maturity *(in progress)*
- [x] Backups scheduled and verified (PBS retention + GC + verify chain)
- [ ] Hardening checklist and review
- [ ] Alerting refinements
- [ ] Rebuild runbook validation

---

## Repository layout
```
README.md              — Overview and index
01-overview/           — Architecture and goals
02-proxmox/            — Proxmox platform (cluster, storage, backups)
03-networking/         — Network services and design (Pi-hole now; OPNsense/VLANs planned)
04-media-stack/        — Media stack documentation
images/diagrams/       — Redacted diagrams and screenshots
```

---

## What this demonstrates

- **Proxmox cluster deployment and management** — multi-node setup, quorum, shared storage, and cluster-level operations
- **Backup architecture and integrity discipline** — PBS with scheduled backups, retention policy, GC, and automated verification (not just backup-and-forget)
- **Service isolation via virtualization** — deliberate VM/CT boundaries rather than co-locating workloads
- **Redundant DNS service design** — dual Pi-hole deployment across separate nodes for availability
- **Storage architecture decisions** — local vs. shared storage, LVM-thin provisioning, NFS for shared ISO access, passthrough for bulk media
- **Secure remote access design** — Tailscale over direct port exposure, ahead of a full edge rollout
- **Incremental network security posture** — DNS filtering now, VLAN segmentation and firewall policy scoped and staged
- **Lab-to-production methodology** — OPNsense staged in a lab VM before promoting to active edge
- **Operational documentation standards** — decisions, current state, and roadmap tracked alongside the build
- **Infrastructure planning discipline** — phased roadmap with clear scope, known gaps acknowledged, and tradeoffs explained
