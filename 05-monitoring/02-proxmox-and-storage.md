# Proxmox and Storage Monitoring

## Goal

Monitor the physical Proxmox hosts, Proxmox Backup Server, cluster health, and storage condition from one monitoring system.

This layer is intended to catch infrastructure problems before they appear only as an application outage.

---

## Physical Host Monitoring

Node Exporter is used on the Proxmox and PBS hosts for operating system metrics such as:

- CPU usage and load
- Memory usage
- Filesystem usage
- Network activity
- Uptime
- Basic host availability

Physical-server dashboards are kept separate from guest dashboards so host pressure is easier to identify.

---

## Proxmox Metrics

The Proxmox API exporter provides cluster-level information that is not available from Node Exporter alone.

This includes visibility into:

- Node state
- VM and LXC state
- Guest resource allocation and usage
- Cluster inventory
- Guest uptime

The API view and the host operating system view are intentionally both retained because they answer different questions.

---

## Storage Health

Storage monitoring includes filesystem capacity together with SMART-related disk health where supported.

The goal is not only to show how full a disk is. Storage monitoring also helps identify aging disks, temperature changes, and drive health indicators that may need review.

The PBS host is included in the same physical-server monitoring model so backup infrastructure is visible alongside the Proxmox nodes.

---

## Operational Use

This layer is useful when troubleshooting questions such as:

- Is a Proxmox node under CPU or memory pressure?
- Is a filesystem filling unexpectedly?
- Is a guest problem actually caused by the host underneath it?
- Is PBS reachable and healthy independently of the scheduled backup job?
- Is a disk showing a condition that should be reviewed before failure?

---

## Validation

Validation includes:

- All Proxmox nodes return Node Exporter metrics
- PBS returns host metrics
- Proxmox API queries return node and guest data
- Storage and SMART collectors remain current
- Grafana panels match the underlying Prometheus data

Older hardware is not treated as a failure by itself. Age and health information are surfaced so they can be reviewed in context.
