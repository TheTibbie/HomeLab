# VM and LXC Monitoring

## Goal

Track the state and resource usage of Proxmox virtual machines and LXC containers without treating guest monitoring as the same thing as physical-host monitoring.

---

## Monitoring Model

Guest visibility comes primarily from the Proxmox API metrics.

This provides a cluster-level view of:

- Guest state
- Guest uptime
- CPU usage
- Memory usage
- Assigned resources
- Guest placement by Proxmox node

The guest inventory is then used in Grafana to show both individual workload health and broader VM/LXC status.

---

## Why Guest Monitoring Is Separate

A running Proxmox node does not guarantee that every guest on it is healthy.

Separating guest state from physical-host state makes it easier to distinguish between:

```text
Host problem
    ↓
Multiple guests may be affected
```

and:

```text
Single guest problem
    ↓
Host remains healthy
```

This is especially useful during maintenance, migration, or service troubleshooting.

---

## Dashboard Use

Guest dashboards are intended to provide quick answers to questions such as:

- Is the VM or container running?
- Which node currently hosts it?
- Has it restarted recently?
- Is CPU or memory usage unusually high?
- Is the problem isolated to one guest or affecting multiple workloads?

The dashboards are kept relatively simple so they remain useful during troubleshooting instead of becoming another inventory page.

---

## Validation

Validation includes:

- Expected VMs and LXCs appear in Prometheus queries
- Guest names and IDs match the Proxmox inventory
- Running/stopped state matches Proxmox
- Uptime values are current
- Node placement is reported correctly
- Grafana guest panels return current data

Guest monitoring supplements application probes. A VM can be running while the service inside it is still unavailable, so both layers are retained.
