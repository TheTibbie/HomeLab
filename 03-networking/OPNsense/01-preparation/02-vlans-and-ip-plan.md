# VLANs and IP Plan

## VLAN definitions

One `/24` per VLAN with OPNsense as the `.1` gateway on each segment:

| VLAN | Name | Subnet | Gateway |
|---|---|---|---|
| 10 | Workstations | `10.40.10.x/24` | `10.40.10.x` |
| 20 | Servers | `10.40.20.x/24` | `10.40.20.x` |
| 30 | IoT | `10.40.30.x/24` | `10.40.30.x` |
| 40 | Guest | `10.40.40.x/24` | `10.40.40.x` |
| 99 | Management | `10.40.99.x/24` | `10.40.99.x` |

---

## Why this scheme

The third octet mirrors the VLAN ID — VLAN 10 is `10.40.10.x`, VLAN 20 is `10.40.20.x`, and so on. This makes firewall rules readable at a glance without needing a reference sheet. The `10.40.x.x` block was chosen specifically to avoid overlapping with the upstream ISP router subnet, which matters for a double-NAT design during the transition period.

VLAN 99 for Management is a deliberate choice — a high, out-of-band ID that's visually distinct and unlikely to be confused with a client-facing segment.

---

## Screenshots

<img width="865" height="509" alt="image" src="https://github.com/user-attachments/assets/8d22a538-4e67-4bb2-bd85-e5ba8ea19e8a" />
