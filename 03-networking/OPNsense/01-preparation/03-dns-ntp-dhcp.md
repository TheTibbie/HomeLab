# DNS, NTP, and DHCP Posture

## DNS — Pi-hole

Pi-hole will live in `VLAN20_SERVERS` once the network is live. Both instances are staged as aliases in OPNsense:

| Instance | Planned IP |
|---|---|
| Pi-hole #1 | `10.40.20.x` |
| Pi-hole #2 | `10.40.20.x` |

Centralizing DNS through Pi-hole enables a "force DNS" firewall policy — traffic attempting to bypass to external resolvers is intercepted and redirected. This prevents untrusted VLANs (IoT, Guest) from using unknown resolvers and keeps DNS visibility intact across the whole network.

---

## NTP — OPNsense as time source

All VLANs are configured to use OPNsense itself as the NTP source (UDP/123 to "This Firewall"). External NTP is not opened broadly.

- Timezone: `America/Toronto`
- NTP status: synced, active peer confirmed during staging

Consistent time across all VLANs matters for log correlation, certificate validation, and authentication. Using OPNsense as the single NTP source also simplifies firewall policy — one rule covers all VLANs rather than opening outbound NTP per segment.

---

## DHCP — Kea (staged, disabled)

Dnsmasq is disabled. Kea DHCPv4 is selected as the long-term DHCP service. Scopes are fully configured for all VLANs but **Kea remains disabled until cutover** — enabling it during staging would create a DHCP conflict with the ISP router on the existing flat LAN.

Staged DHCP pools:

| VLAN | Pool |
|---|---|
| VLAN 10 — Workstations | `10.40.10.x–10.40.10.x` |
| VLAN 20 — Servers | `10.40.20.x–10.40.20.x` |
| VLAN 30 — IoT | `10.40.30.x–10.40.30.x` |
| VLAN 40 — Guest | `10.40.40.x–10.40.40.x` |
| VLAN 99 — Management | `10.40.99.x–10.40.99.x` (small convenience pool) |

Low and high ranges on each subnet are reserved for static assignments and infrastructure — DHCP pools sit in the middle. The consistent range structure across VLANs makes the addressing scheme predictable and easy to read in firewall rules.

---

## Screenshots

<img width="859" height="551" alt="image" src="https://github.com/user-attachments/assets/4cae5b08-4cb9-4113-9c43-ad6dad00c008" />
