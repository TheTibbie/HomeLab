# Core Monitoring

## Goal

Provide one central place to collect infrastructure metrics and display the overall health of the homelab.

Prometheus is used as the primary metrics store and Grafana is used for dashboards, troubleshooting views, and alerting.

---

## Core Components

| Component | Role |
|---|---|
| Prometheus | Metrics collection and query engine |
| Grafana | Dashboards and alerting |
| Node Exporter | Linux host operating system metrics |
| Blackbox Exporter | HTTP and TCP-style endpoint checks |
| Proxmox exporter | Proxmox cluster and guest metrics |

The core monitoring services are hosted on the observability VM rather than being distributed across the Proxmox nodes.

---

## Collection Model

Prometheus scrapes a mixture of exporters and custom collectors.

The design keeps raw collection separate from visualization:

```text
Host / Service
      ↓
Exporter or Collector
      ↓
Prometheus
      ↓
Grafana
```

This makes it possible to test each part independently when a dashboard appears incorrect.

---

## Dashboard Approach

Grafana contains both detailed dashboards and a higher-level HomeLab Overview dashboard.

The overview is intended to answer a simpler question first: is the lab healthy, and if not, which layer needs investigation?

More detailed dashboards are then used for host, storage, guest, Docker, and application-level troubleshooting.

---

## Validation

Core validation includes:

- Prometheus target status is healthy
- Exporter endpoints return metrics
- Grafana can query Prometheus successfully
- Dashboard panels return current data
- Blackbox probes return expected success values
- Monitoring data remains current after infrastructure changes

The raw Prometheus and Grafana query views remain useful during troubleshooting, but the main dashboards are the normal day-to-day view.
