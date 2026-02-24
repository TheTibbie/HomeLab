# Guest Inventory

## Redaction note

LAN IPs are shown as `192.168.x.x`. Resource allocations are included to communicate sizing intent, not as deployment prescriptions.

---

| ID | Guest | Type | Node | Resources | LAN IP | Purpose | Backup |
|---|---|---|---|---|---|---|---|
| 100 | `media-vm` | VM | `proxmox-01` | 4 vCPU, 8 GiB RAM, 120G disk | — | Media stack with bulk disk passthrough | PBS |
| 101 | `pihole` | CT | `proxmox-02` | 1 vCPU, 1 GiB RAM, 8G disk | `192.168.x.x` | DNS filtering — primary | PBS |
| 102 | `pihole-b` | CT | `proxmox-03` | 1 vCPU, 1 GiB RAM, 8G disk | `192.168.x.x` | DNS filtering — secondary | PBS |
| 103 | `uptime-kuma` | CT | `proxmox-02` | 1 vCPU, 1 GiB RAM, 8G disk | `192.168.x.x` | Availability monitoring | PBS |
| 104 | `opnsense-lab` | VM | `proxmox-04` | 2 vCPU, 4 GiB RAM, 32G disk | `192.168.x.x` | OPNsense lab VM — not active edge | PBS |
| 105 | `homeassistant` | VM | `proxmox-02` | 2 vCPU, 4 GiB RAM, 32G disk | `192.168.x.x` | Home automation | PBS |
| 106 | `isponsorblocktv` | CT | `proxmox-02` | 1 vCPU, 512 MiB RAM, 8G disk | `192.168.x.x` | SponsorBlock utility for smart TV | PBS |
| 120 | `dashy` | CT | `proxmox-01` | 2 vCPU, 2 GiB RAM, 8G disk | `192.168.x.x` | Service dashboard | PBS |

---

## Notes

- ISP router remains the active gateway and DHCP server
- `opnsense-lab` (VM 104) is in staging — dedicated dual-NIC edge deployment is the next planned phase
- `media-vm` (VM 100) has no static LAN IP listed; network access is via the bulk disk passthrough configuration [TODO: confirm if media-vm has a LAN IP or is access handled differently]
