# Firewall Policy by VLAN (Staged)

## Overall intent

- Default deny between VLANs
- Permit only what is explicitly required
- Force DNS to Pi-hole and NTP to OPNsense on all segments
- Management is the most restricted segment — inbound access is tightly controlled

---

## VLAN 10 — Workstations

| # | Action | Traffic |
|---|---|---|
| 1 | Allow | NTP → OPNsense (UDP `NTP_PORT`) |
| 2 | Allow | DNS → `PIHOLES` (TCP/UDP `DNS_PORTS`) |
| 3 | Block | DNS → anywhere else (TCP/UDP `DNS_PORTS`) |
| 4 | Allow | Workstations → Management (TCP `MGMT_ADMIN_PORTS`) |
| 5 | Block | Workstations → `UNTRUSTED_VLANS` |
| 6 | Allow | Workstations → Internet (any) |

Workstations get full internet access for daily use. DNS is forced through Pi-hole — rule 3 blocks any attempt to bypass to an external resolver. IoT and Guest are blocked by default; Management is reachable only on defined admin ports.

---

## VLAN 20 — Servers

| # | Action | Traffic |
|---|---|---|
| 1 | Allow | DNS → `PIHOLES` (TCP/UDP `DNS_PORTS`) |
| 2 | Block | DNS → anywhere else |
| 3 | Allow | NTP → OPNsense (UDP `NTP_PORT`) |
| 4 | Block | Servers → Management (any to VLAN 99) |
| 5 | Allow | Servers → Internet (any) |

Servers need outbound internet for updates and functionality. DNS is forced. The explicit block to Management prevents lateral movement from a compromised server into the management segment.

---

## VLAN 30 — IoT

| # | Action | Traffic |
|---|---|---|
| 1 | Allow | IoT → `MEDIA_SERVER` (TCP `JELLYFIN_PORT`) |
| 2 | Allow | DNS → `PIHOLES` (TCP/UDP `DNS_PORTS`) |
| 3 | Block | DNS → anywhere else |
| 4 | Allow | NTP → OPNsense (UDP `NTP_PORT`) |
| 5 | Block | IoT → `INTERNAL_VLANS` |
| 6 | Allow | IoT → Internet (TCP `WEB_PORTS`) |

IoT gets the most restrictive outbound policy. The single inter-VLAN exception is Jellyfin on the media server — IoT devices (smart TVs, etc.) need to reach it. Rule 5 blocks everything else internal. Outbound internet is limited to web ports only, covering firmware updates without opening broad access.

---

## VLAN 40 — Guest

| # | Action | Traffic |
|---|---|---|
| 1 | Allow | DNS → `PIHOLES` (TCP/UDP `DNS_PORTS`) |
| 2 | Block | DNS → anywhere else |
| 3 | Allow | NTP → OPNsense (UDP `NTP_PORT`) |
| 4 | Block | Guest → `INTERNAL_VLANS` |
| 5 | Allow | Guest → Internet (any) |

Guest gets internet access with no visibility into internal networks. DNS cannot bypass Pi-hole. The segment is otherwise fully isolated from the rest of the environment.

---

## VLAN 99 — Management

| # | Action | Traffic |
|---|---|---|
| 1 | Allow | DNS → `PIHOLES` (TCP/UDP `DNS_PORTS`) |
| 2 | Block | DNS → anywhere else |
| 3 | Allow | NTP → OPNsense (UDP `NTP_PORT`) |
| 4 | Allow | MGMT → Internet (TCP `WEB_PORTS`) |

Management outbound is web-only — enough for updates and downloads without opening broad internet access. No inbound rules from other VLANs are defined here; access to Management from Workstations is handled at the Workstations ruleset (rule 4 above).

---

## Screenshots

<img width="859" height="442" alt="image" src="https://github.com/user-attachments/assets/18bc7c67-79c4-4bbf-b079-068f6198848a" />

<img width="872" height="413" alt="image" src="https://github.com/user-attachments/assets/75fdffd6-e42f-4a9b-91db-522642421d9a" />
