# Backup Coverage

## Goal

Capture enough configuration and recovery information to rebuild the important homelab infrastructure without storing the actual backup payloads in GitHub.

---

## Infrastructure Coverage

| Component | Backup Method |
|---|---|
| Proxmox nodes | Host configuration archives |
| Proxmox cluster | Encrypted cluster configuration archive |
| Proxmox Backup Server | Host configuration + encrypted application configuration |
| OPNsense | Native encrypted configuration export |
| Omada Controller | Native controller export |
| Pi-hole primary | Teleporter export |
| Pi-hole secondary | Teleporter export |
| Observability VM | Configuration archive + encrypted sensitive archive |
| Uptime Kuma | Application state and Compose configuration |
| Home Assistant | Native encrypted backup |
| Dashy | Configuration and Compose files |
| Reverse proxy | Caddy configuration + PBS guest backup |

Supporting scripts, service units, and host networking configuration were included where they are required to rebuild the related service correctly.

---

## Proxmox and PBS

Each Proxmox node has a host-level configuration archive containing the important networking, hostname, filesystem, boot, and service configuration required during recovery.

A separate encrypted backup of `/etc/pve` captures the cluster-level configuration, including guest definitions, storage configuration, backup jobs, Corosync, firewall configuration, and credential-bearing cluster data.

PBS follows the same model:

- Host operating system configuration
- Separate encrypted application configuration from `/etc/proxmox-backup`

This is separate from the actual guest backup datastore.

---

## Network Infrastructure

OPNsense and Omada use their native export functions.

Both Pi-hole instances use Teleporter exports so local DNS records and Pi-hole configuration can be restored independently.

The reverse proxy also has a standalone copy of the active Caddy configuration in addition to the normal PBS backup of the LXC.

---

## Monitoring and Supporting Services

The observability backup focuses on rebuild capability rather than long-term telemetry history.

Configuration for Prometheus, Grafana provisioning, Loki, Alloy, Blackbox Exporter, exporters, and supporting scripts is preserved.

Selected sensitive monitoring data is kept in a separate encrypted archive.

Uptime Kuma, Home Assistant, and Dashy use application-appropriate backup methods so their important state can be restored without rebuilding configuration manually.

---

## Intentional Boundary

This repository documents what is protected and the recovery approach.

It does not contain:

- Backup archives
- Application databases
- Private keys
- Passwords or tokens
- Encrypted backup payloads
- Password-manager contents

Those remain in the private off-host backup location.
