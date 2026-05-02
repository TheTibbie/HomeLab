# Uptime Kuma (Monitoring)

## Summary

Uptime Kuma provides lightweight availability monitoring across core homelab services. It is treated as a baseline operational capability for visibility during change and expansion, not an afterthought.

After the OPNsense VLAN cutover, Uptime Kuma is placed on the Management / Servers VLAN with other infrastructure services. Monitoring targets were updated after service migration so they reflect the current network layout.

---

## Deployment

| Detail | Value |
|---|---|
| Guest | CT 103 (`uptime-kuma`) |
| Node | `proxmox-02` |
| Access | Management / Servers VLAN |

Full IP addresses and service URLs are intentionally redacted.

---

## Post-Cutover Role

Uptime Kuma was updated after the network migration because multiple monitored services moved from the old flat network to the new VLAN layout.

It is used as the primary service-health view for:

- Core infrastructure services
- Network services
- Dashboards
- Self-hosted applications
- Migrated workloads after the OPNsense cutover

Dashy is treated as a convenience dashboard. Uptime Kuma is the stronger source of truth for whether services are reachable.

---

![Uptime Kuma dashboard showing monitored homelab services](https://github.com/user-attachments/assets/a2213ff9-d87c-49f1-8bbf-1e341c88e069)
