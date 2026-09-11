# Alerting

## Goal

Turn the monitoring stack into something that can surface important problems without requiring the dashboards to be watched continuously.

---

## Grafana Alerting

Grafana is used for selected infrastructure alerts where a clear condition can be defined from Prometheus data.

Alerts are kept focused on meaningful conditions rather than every temporary metric change.

Examples include:

- Host or service availability problems
- Monitoring target failures
- Storage or filesystem conditions that require review
- Monitoring-system health conditions

Telegram is used as the notification path for selected Grafana alerts.

---

## Independent Watchdog

Uptime Kuma is retained as an independent monitoring path.

Its role is not to duplicate every Prometheus metric. It provides a simpler external check that can still identify a problem if the main Prometheus/Grafana stack itself is unhealthy.

This creates a useful separation:

```text
Prometheus / Grafana
→ Detailed internal monitoring

Uptime Kuma
→ Independent availability check
```

---

## HomeLab Overview

The HomeLab Overview dashboard provides a high-level operational view of the environment.

It combines selected physical-host, service, storage, monitoring, and reverse-proxy status into one dashboard so the first troubleshooting step does not require opening multiple dashboards.

The overview is intentionally not a replacement for the detailed dashboards.

---

## Alerting Approach

The main goal is actionable alerting.

A condition is more useful as an alert when it indicates something that should actually be investigated, rather than a normal short-lived fluctuation.

For that reason:

- Not every dashboard panel has an alert
- Warning conditions are separated from hard failures where practical
- The monitoring stack itself is monitored
- Independent checks remain available outside Grafana

---

## Validation

Alerting validation includes:

- Grafana alert rules evaluate successfully
- Notification delivery works
- Recovery states clear correctly
- Uptime Kuma continues to check the monitoring system independently
- HomeLab Overview status matches the underlying monitoring data

Alert thresholds continue to be adjusted as the environment develops and normal operating ranges become clearer.
