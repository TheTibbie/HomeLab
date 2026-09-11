# Centralized Logging

## Goal

Collect useful operational logs in one place so infrastructure problems can be reviewed alongside metrics instead of logging into each host individually.

---

## Logging Stack

| Component | Role |
|---|---|
| Loki | Central log storage and query backend |
| Alloy | Log collection and forwarding |
| Grafana | Log search and correlation with metrics |

The logging layer was added after the core metrics environment was already working.

---

## What Is Collected

The logging configuration focuses on operationally useful data rather than attempting to ingest every available log.

Examples include:

- Linux system and service logs
- Proxmox and PBS host logs
- Kernel and storage-related events
- Selected infrastructure service logs
- Monitoring-stack logs where useful for troubleshooting

The goal is to make meaningful failures easier to find without creating unnecessary log volume.

---

## Why Logs Are Kept Separate From Metrics

Metrics are useful for showing that something changed.

Logs are often more useful for showing why it changed.

The two layers are used together:

```text
Grafana metric or alert shows a problem
                ↓
Review related Loki logs
                ↓
Identify service, kernel, storage, or network evidence
```

This is especially useful for short-lived errors that may be gone by the time the system is checked manually.

---

## Log Rotation and Retention

Local logging and collector configuration were reviewed so monitoring additions do not create uncontrolled disk growth.

Where custom monitoring scripts or services produce logs, rotation is used where appropriate rather than allowing files to grow indefinitely.

---

## Validation

Validation includes:

- Alloy is active and forwarding expected logs
- Loki is reachable from Grafana
- Recent host logs can be queried
- Labels identify the correct host or service
- Storage and kernel events can be found when present
- Logging does not interfere with normal host operation

A lack of matching log events is not treated as a failure by itself. Collector and service health are checked separately.
