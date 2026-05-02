# Pi-hole (DNS Filtering)

## Summary

Two Pi-hole instances run as LXC containers on separate Proxmox nodes, providing DNS filtering with basic redundancy.

After the OPNsense VLAN cutover, DHCP is handled by OPNsense instead of the ISP router. OPNsense distributes both Pi-hole instances as DNS servers to client VLANs.

The Pi-hole containers are currently placed on the Management / Servers VLAN with other core infrastructure services.

---

## Instances

| Guest | Node | Management UI |
|---|---|---|
| CT 101 (`pihole`) | `proxmox-02` | Management / Servers VLAN |
| CT 102 (`pihole-b`) | `proxmox-03` | Management / Servers VLAN |

Full IP addresses are intentionally redacted.

---

## DNS and DHCP Posture

Current posture after the OPNsense cutover:

- DHCP is handled by OPNsense
- OPNsense distributes both Pi-hole instances as DNS servers
- Pi-hole provides DNS filtering for client VLANs
- Upstream DNS for both Pi-hole instances is Google
- Pi-hole hosts are allowed through firewall policy to reach upstream DNS
- Client DNS bypass is restricted where DNS enforcement is enabled

Placing the two instances on separate Proxmox nodes means a single node failure does not take out both DNS filtering instances.

---

## Network Placement

The original flat LAN design placed both Pi-hole containers on the old LAN while DHCP was still handled by the ISP router.

After the OPNsense cutover, Pi-hole was moved into the Management / Servers VLAN. This matches the current implementation where core infrastructure services are consolidated there until the dedicated server VLAN is reintroduced later.

The dedicated server VLAN remains staged for future use, but it is not currently the active placement model for Pi-hole.

---

## Firewall Consideration

Pi-hole requires two different DNS paths:

- Client VLANs need to query Pi-hole
- Pi-hole itself needs to query upstream DNS resolvers

During the OPNsense cutover, firewall policy had to allow the Pi-hole hosts to reach upstream DNS while still preventing normal clients from bypassing Pi-hole where enforcement is enabled.

This keeps DNS visibility centralized without breaking Pi-hole resolution.

---

## Backups

Both containers are included in the Proxmox backup plan. See [PBS weekly backups](../../02-proxmox/pbs-weekly-backups/).

---

![Pi-hole primary instance dashboard](https://github.com/user-attachments/assets/cbf25f08-2bc4-47b2-b704-9f0b884a2cec)

![Pi-hole secondary instance dashboard](https://github.com/user-attachments/assets/c340dab6-288a-48f6-89ba-150acc442046)

---

## Pages

- [Inventory and deployment notes](01-inventory-and-deployment.md)
- [DNS design and redundancy approach](02-dns-and-redundancy.md)
- [Operations and backups](03-operations-and-backups.md)
