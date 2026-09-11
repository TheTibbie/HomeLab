# Monitoring and Observability

## Summary

This section documents the monitoring stack used to track the health of the homelab across physical hosts, Proxmox, virtual guests, containers, services, logs, and alerts.

The monitoring environment was built in phases instead of as one large deployment. Each phase added another layer while keeping the previous checks in place.

---

## Current Monitoring Stack

The current stack includes:

- Prometheus for metrics collection
- Grafana for dashboards and alerting
- Node Exporter for Linux host metrics
- Proxmox API metrics for cluster and guest visibility
- SMART and storage-related monitoring
- cAdvisor for Docker container metrics
- Blackbox Exporter for HTTP and service-path checks
- Loki and Alloy for centralized log collection
- Pi-hole health and application-specific collectors where useful
- Uptime Kuma as an independent availability check
- Telegram notifications for selected Grafana alerts

---

## Monitoring Layers

| Layer | Purpose |
|---|---|
| Core monitoring | Prometheus, Grafana, exporters, and basic target health |
| Proxmox and storage | Physical nodes, PBS, cluster metrics, SMART, and storage health |
| VM and LXC monitoring | Guest uptime, state, resource visibility, and inventory |
| Docker monitoring | Container CPU, memory, networking, and restart visibility |
| Application monitoring | HTTP/TCP probes and selected application health collectors |
| Logging | Loki and Alloy for centralized operational logs |
| Alerting | Grafana alerts, Telegram notifications, and independent watchdog checks |

---

## Pages

- [Core monitoring](01-core-monitoring.md)
- [Proxmox and storage monitoring](02-proxmox-and-storage.md)
- [VM and container monitoring](03-vm-and-container-monitoring.md)
- [Docker monitoring](04-docker-monitoring.md)
- [Application monitoring](05-application-monitoring.md)
- [Logging](06-logging.md)
- [Alerting](07-alerting.md)

---

## Current Status

The main monitoring build is complete and operational.

The final design intentionally uses more than one monitoring path. Prometheus and Grafana provide the detailed internal view, while Uptime Kuma remains useful as a simpler independent check that can still show a problem if the main monitoring stack itself is unhealthy.

Monitoring continues to be adjusted as new infrastructure is added to the lab.
