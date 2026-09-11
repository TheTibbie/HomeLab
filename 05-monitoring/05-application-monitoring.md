# Application Monitoring

## Goal

Monitor whether important services are actually usable, not just whether the VM, container, or host underneath them is running.

---

## Probe Model

Blackbox Exporter is used for selected HTTP and network-path checks.

The general model is:

```text
Prometheus
    ↓
Blackbox Exporter
    ↓
Service endpoint
```

This provides a simple success/failure signal for service paths that matter operationally.

---

## Direct Checks and End-to-End Checks

Most service probes continue to use direct backend addresses.

That is intentional because a direct check answers whether the application itself is reachable.

The internal DNS / reverse proxy project added a separate end-to-end probe for the reverse proxy path.

This creates two useful troubleshooting signals:

```text
Backend healthy + reverse proxy unhealthy
→ investigate DNS, Caddy, or the proxy path

Backend unhealthy
→ investigate the application, guest, or underlying host
```

The existing backend probes were not replaced with DNS names.

---

## Pi-hole Monitoring

Pi-hole monitoring includes both normal service checks and custom metrics where useful.

The monitoring design checks collector success and metric freshness so a stale collector does not look healthy simply because an old metric still exists in Prometheus.

Both Pi-hole instances are monitored separately because they provide redundant DNS service.

---

## Reverse Proxy Monitoring

The reverse proxy has a dedicated lightweight health endpoint under the internal `home.arpa` namespace.

A Blackbox probe checks that DNS resolution and the Caddy HTTP path are both working.

The resulting probe is also displayed on the HomeLab Overview dashboard.

---

## Validation

Application monitoring is validated by checking:

- Direct service probes return the expected result
- Blackbox targets resolve correctly
- Probe success values are current
- Custom collectors report successful collection
- Collector freshness remains within the expected interval
- Grafana panels match the underlying Prometheus query

Application probes supplement infrastructure monitoring. They are not used as a replacement for host, guest, or container metrics.
