# Pi-hole (DNS Filtering)

## Summary

Two Pi-hole instances run as LXC containers on separate Proxmox nodes, providing DNS filtering with basic redundancy. If one instance is unavailable, the ISP router falls back to the second automatically.

---

## Instances

| Guest | Node | LAN UI |
|---|---|---|
| CT 101 (`pihole`) | `proxmox-02` | `http://192.168.x.x/admin/` |
| CT 102 (`pihole-b`) | `proxmox-03` | `http://192.168.x.x/admin/` |

---

## DNS and DHCP posture

- DHCP is handled by the ISP router
- The ISP router distributes both Pi-hole instances as DNS servers to LAN clients
- Upstream DNS for both instances: Google

Placing the two instances on separate nodes (`proxmox-02` and `proxmox-03`) means a single node failure doesn't take out DNS for the whole network.

---

## Backups

Both containers are included in the weekly PBS backup job. See [PBS weekly backups](../../02-proxmox/pbs-weekly-backups/).

---

![Pi-hole primary instance dashboard](https://github.com/user-attachments/assets/cbf25f08-2bc4-47b2-b704-9f0b884a2cec)

![Pi-hole secondary instance dashboard](https://github.com/user-attachments/assets/c340dab6-288a-48f6-89ba-150acc442046)

---

## Pages

- [Inventory and deployment notes](01-inventory-and-deployment.md)
- [DNS design and redundancy approach](02-dns-and-redundancy.md)
- [Operations and backups](03-operations-and-backups.md)
