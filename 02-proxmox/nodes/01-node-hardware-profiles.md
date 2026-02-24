# Node Hardware Profiles

## Redaction note

Internal IPs are shown as `192.168.x.x`. Disk sizes are approximate and included to communicate capacity intent.

---

## Cluster nodes

| Detail | `proxmox-01` | `proxmox-02` | `proxmox-03` | `proxmox-04` |
|---|---|---|---|---|
| CPU | i5-6500 (4C/4T) | i5-4570 (4C/4T) | i5-4570 (4C/4T) | i5-4570 (4C/4T) |
| RAM | ~15 GiB | ~11 GiB | ~11 GiB | ~11 GiB |
| Swap | 8 GiB | ~7.6 GiB | ~7.6 GiB | 8 GiB |
| Primary disk | ~476.9G SSD | ~447.1G SSD | ~447.1G SSD | ~447.1G SSD |
| Storage type | LVM2 + LVM-thin | LVM2 + LVM-thin | LVM2 + LVM-thin | LVM2 + LVM-thin |
| NIC | Realtek RTL8111/8168 | Intel I217-LM | Intel I217-LM | Intel I217-LM |
| Bridge | `vmbr0` | `vmbr0` | `vmbr0` | `vmbr0` |
| LAN IP | `192.168.x.x/24` | `192.168.x.x/24` | `192.168.x.x/24` | `192.168.x.x/24` |
| Additional storage | ~4.5T WD (ext4, `media-drive`) | — | — | — |

---

## Node roles

**`proxmox-01`** — Hosts the media VM and dashboard services. The additional WD drive is passed through directly to the media VM for bulk storage; it doesn't participate in Proxmox-managed storage.

**`proxmox-02`** — Runs core infrastructure containers: primary DNS (Pi-hole), availability monitoring (Uptime Kuma), and utility services.

**`proxmox-03`** — Dedicated to the secondary Pi-hole instance. Keeping DNS redundancy on a separate node ensures a single node failure doesn't take out both resolvers.

**`proxmox-04`** — Currently hosts the OPNsense lab VM while dedicated edge hardware is being planned. The node is otherwise lightly loaded, which makes it a natural staging environment for the networking phase.

<img width="335" height="196" alt="image" src="https://github.com/user-attachments/assets/5867599f-ddb0-417b-b4ec-ec0c244c4560" />

