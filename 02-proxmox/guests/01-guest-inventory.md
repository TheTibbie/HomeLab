# Guest Inventory

## Redaction Note

Internal IPs are redacted or generalized. Resource allocations are included to communicate sizing intent, not as deployment prescriptions.

After the OPNsense VLAN cutover, infrastructure guests are placed on the Management / Servers VLAN unless otherwise noted.

---

| ID | Guest | Type | Node | Resources | Network Placement | Purpose | Backup |
|---|---|---|---|---|---|---|---|
| 100 | `media-vm` | VM | `proxmox-01` | 4 vCPU, 8 GiB RAM, 120G disk | Management / Servers VLAN | Media stack with bulk disk passthrough | PBS |
| 101 | `pihole` | CT | `proxmox-02` | 1 vCPU, 1 GiB RAM, 8G disk | Management / Servers VLAN | DNS filtering, primary instance | PBS |
| 102 | `pihole-b` | CT | `proxmox-03` | 1 vCPU, 1 GiB RAM, 8G disk | Management / Servers VLAN | DNS filtering, secondary instance | PBS |
| 103 | `uptime-kuma` | CT | `proxmox-02` | 1 vCPU, 1 GiB RAM, 8G disk | Management / Servers VLAN | Availability monitoring | PBS |
| 104 | `opnsense-lab` | VM | `proxmox-04` | 2 vCPU, 4 GiB RAM, 32G disk | Active edge / firewall | OPNsense router, firewall, DHCP, and VLAN gateway | PBS |
| 105 | `homeassistant` | VM | `proxmox-02` | 2 vCPU, 4 GiB RAM, 32G disk | Management / Servers VLAN | Home automation | PBS |
| 106 | `isponsorblocktv` | CT | `proxmox-02` | 1 vCPU, 512 MiB RAM, 8G disk | Deferred / not active cutover scope | SponsorBlock utility for smart TV | PBS |
| 120 | `dashy` | CT | `proxmox-01` | 2 vCPU, 2 GiB RAM, 8G disk | Management / Servers VLAN | Service dashboard | PBS |

---

## Notes

- OPNsense is now the active router, firewall, and DHCP service for the environment.
- The ISP router is no longer the primary internal gateway/DHCP authority for the lab network.
- `opnsense-lab` (VM 104) started as the staged OPNsense VM and now functions as the active edge for the VLAN design.
- Several infrastructure and service guests are consolidated into the Management / Servers VLAN because part of the Proxmox infrastructure still sits behind an unmanaged switch.
- The dedicated server VLAN remains staged for future use when the switching layer supports cleaner workload separation.
- `media-vm` (VM 100) uses bulk disk passthrough for media storage and is documented separately in the storage and media sections.
