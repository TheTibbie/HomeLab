# Node Hardware Profiles

## Redaction Note

Internal IPs are redacted or generalized. Disk sizes are approximate and included to communicate capacity intent.

After the OPNsense VLAN cutover, Proxmox management addresses are on the Management / Servers VLAN.

---

## Cluster Nodes

| Detail | `proxmox-01` | `proxmox-02` | `proxmox-03` | `proxmox-04` |
|---|---|---|---|---|
| CPU | i5-6500 (4C/4T) | i5-4570 (4C/4T) | i5-4570 (4C/4T) | i5-4570 (4C/4T) |
| RAM | ~15 GiB | ~11 GiB | ~11 GiB | ~11 GiB |
| Swap | 8 GiB | ~7.6 GiB | ~7.6 GiB | 8 GiB |
| Primary disk | ~476.9G SSD | ~447.1G SSD | ~447.1G SSD | ~447.1G SSD |
| Storage type | LVM2 + LVM-thin | LVM2 + LVM-thin | LVM2 + LVM-thin | LVM2 + LVM-thin |
| NIC | Realtek RTL8111/8168 | Intel I217-LM | Intel I217-LM | Intel I217-LM |
| Bridge | `vmbr0` | `vmbr0` | `vmbr0` | `vmbr0` |
| Management network | Management / Servers VLAN | Management / Servers VLAN | Management / Servers VLAN | Management / Servers VLAN |
| Additional storage | ~4.5T WD (ext4, `media-drive`) | None | None | None |

---

## Node Roles

**`proxmox-01`**

Hosts the media VM and dashboard services. The additional WD drive is passed through directly to the media VM for bulk storage. It does not participate in Proxmox-managed storage.

**`proxmox-02`**

Runs core infrastructure containers and services, including the primary Pi-hole instance, Uptime Kuma, Home Assistant, and supporting utility workloads.

**`proxmox-03`**

Hosts the secondary Pi-hole instance and the Active Directory domain controller lab. Keeping DNS redundancy on a separate node ensures a single node failure does not take out both Pi-hole resolvers.

**`proxmox-04`**

Hosts the active OPNsense VM. This node carries an important infrastructure role because OPNsense provides routing, firewall policy, DHCP, VLAN gateways, and inter-VLAN control for the environment.

Because this node hosts the active edge VM, network changes on `proxmox-04` require extra caution. A misconfiguration can affect routing, management access, and recovery paths for the rest of the lab.

---

## Post-Cutover Networking Note

Before the OPNsense VLAN cutover, the Proxmox nodes were managed on the old flat network.

After the cutover, Proxmox management moved to the Management / Servers VLAN. Corosync ring addresses were also updated so the cluster could regain quorum after the network migration.

The cutover and recovery process is documented in:

`../../03-networking/OPNsense/02-cutover-and-implementation/05-issues-encountered-and-resolutions.md`

---

<img width="335" height="196" alt="Proxmox cluster node hardware summary" src="https://github.com/user-attachments/assets/5867599f-ddb0-417b-b4ec-ec0c244c4560" />
