# Docker Monitoring

## Goal

Add container-level visibility for Docker workloads without replacing the existing host and application monitoring layers.

---

## cAdvisor

cAdvisor is used to expose Docker container metrics to Prometheus.

This adds visibility into:

- Container CPU usage
- Container memory usage
- Container network activity
- Container runtime state
- Container start time and restart behavior

Docker monitoring is useful when the host itself is healthy but one container is consuming unusual resources or restarting unexpectedly.

---

## Monitoring Path

```text
Docker Engine
    ↓
cAdvisor
    ↓
Prometheus
    ↓
Grafana
```

The cAdvisor layer is intentionally kept separate from the application health layer.

A container can be running while the application inside it is not responding correctly. For that reason, container metrics and application probes are both retained.

---

## Restart Visibility

Container start-time metrics are used to help identify recent restarts.

This is useful when reviewing intermittent issues where a service appears healthy by the time it is checked but may have restarted earlier.

Restart information is treated as troubleshooting context rather than an automatic failure condition by itself.

---

## Dashboard Use

The Docker dashboard is mainly used for:

- Comparing resource usage between containers
- Spotting a container using significantly more memory or CPU than expected
- Reviewing network activity
- Checking whether a container has restarted
- Confirming the container runtime layer is healthy before moving deeper into application troubleshooting

---

## Validation

Validation includes:

- cAdvisor is reachable by Prometheus
- Expected containers appear in metrics
- Container names are populated correctly
- CPU and memory data update over time
- Start-time metrics are available
- Grafana panels match the Prometheus data

Docker monitoring remains one layer of the overall monitoring system rather than the only indication of service health.
