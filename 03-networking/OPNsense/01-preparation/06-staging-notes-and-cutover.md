# Staging Notes and Cutover Plan

> **Historical note:** This document reflects the original pre-cutover staging plan. The final implementation and any changes made during the actual cutover are documented in `../02-cutover-and-implementation/`.
> 
## Staging notes

**Temporary WAN during staging** — OPNsense was briefly given a default route via the existing ISP router to pull updates, then disabled again. WAN has otherwise remained down throughout staging to avoid impacting the live LAN.

**Kea DHCP** — Fully configured for all VLANs but remains disabled until cutover. Enabling it before the switch is in place would create a DHCP conflict with the ISP router.

**WAN — Block private networks** — Disabled on the WAN interface. Required for the double NAT design: OPNsense's WAN will receive a private RFC1918 IP from the ISP router, and leaving this enabled would drop return traffic and break internet connectivity at cutover. This was addressed during staging.

---

## Cutover steps

1. Export an OPNsense config backup as a post-staging snapshot
2. Confirm final switch and AP selection — determine trunk port configuration and native VLAN approach
3. Move Pi-hole instances into VLAN 20 and assign static IPs (`10.40.20.x` and `10.40.20.x`)
4. At cutover:
   - Disable DHCP on the ISP router
   - Enable Kea DHCPv4 in OPNsense
   - Confirm trunk port carries VLANs 10/20/30/40/99
   - Map SSIDs to VLANs (Workstations, IoT, Guest)

---

## Post-cutover validation

**Per VLAN — confirm each receives:**
- Correct DHCP lease
- Correct gateway
- DNS resolving via Pi-hole

**Cross-VLAN policy — confirm:**
- Guest cannot reach internal VLANs
- IoT can reach the permitted internal service (`MEDIA_SERVER`) but nothing else internal
- Workstations can reach Management interfaces on admin ports only

---
