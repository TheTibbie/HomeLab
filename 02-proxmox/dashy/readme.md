# Dashy (Service Dashboard)

## Summary

Dashy provides a single landing page for accessing internal homelab services, including hypervisor UIs, PBS, DNS, monitoring, and anything else that needs a persistent bookmark. It is a usability layer, not infrastructure, but it reduces friction during day-to-day operations and service validation across nodes.

After the OPNsense VLAN cutover, Dashy is placed on the Management / Servers VLAN with other infrastructure services. Dashboard links should continue to be reviewed as services move or receive updated addresses.

---

## Deployment

| Detail | Value |
|---|---|
| Guest | CT 120 (`dashy`) |
| Node | `proxmox-01` |
| Access | Management / Servers VLAN |

Full IP addresses and service URLs are intentionally redacted.

---

## Post-Cutover Role

Dashy was updated after the network migration so it continues pointing to current service locations.

It is used as a convenience dashboard for:

- Proxmox node UIs
- Proxmox Backup Server
- Pi-hole
- Uptime Kuma
- OPNsense
- Omada
- Self-hosted applications

Dashy is not treated as the source of truth for service health. Uptime Kuma remains the better validation point for whether services are reachable.

---

![Dashy dashboard showing homelab service links](https://github.com/user-attachments/assets/dfec81ef-e402-4755-801c-4df0f3fc6565)
