# Firewall Aliases (Maintainability)

## Why aliases

Aliases keep firewall rules readable and maintainable. Rather than hardcoding IPs and ports directly into rules, aliases are defined once and referenced by name — updating a service address or port means editing one alias, not hunting through every rule that references it.

---

## Host aliases

| Alias | Value |
|---|---|
| `PIHOLE_1` | `10.40.20.x` |
| `PIHOLE_2` | `10.40.20.x` |
| `PIHOLES` | `10.40.20.x, 10.40.20.x` |
| `MEDIA_SERVER` | `10.40.20.x` |

---

## Port aliases

| Alias | Ports |
|---|---|
| `DNS_PORTS` | `53, 853` |
| `NTP_PORT` | `123` |
| `WEB_PORTS` | `80, 443` |
| `JELLYFIN_PORT` | `8096` |
| `MGMT_ADMIN_PORTS` | `443, 8006, 22` |

---

## Network aliases

| Alias | Subnets |
|---|---|
| `INTERNAL_VLANS` | `10.40.10.x/24, 10.40.20.x/24, 10.40.30.x/24, 10.40.99.x/24` |
| `UNTRUSTED_VLANS` | `10.40.30.x/24, 10.40.40.x/24` |

`UNTRUSTED_VLANS` groups IoT and Guest — the two segments that get the most restrictive outbound policy and no inter-VLAN access by default.

---

## Screenshots

<img width="981" height="1035" alt="image" src="https://github.com/user-attachments/assets/1dae8d2c-0dcf-4f68-9a94-fbc55e69c643" />
