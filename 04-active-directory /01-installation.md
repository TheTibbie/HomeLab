# Phase 1 - VM Creation & OS Installation

## Overview

Before anything was built, the environment was scoped out. Proxmox handles the DC, VirtualBox on the main Windows PC handles the clients. The domain is `exodus.lab`, one DC and two Windows 11 clients. DHCP was kept off DC01 from the start - in production, DHCP lives on network infrastructure, not domain controllers, and the lab is built to reflect that.

---

## DC01 VM Configuration

DC01 was created on `proxmox-03` with the following spec:

| Setting | Value | Reason |
|---|---|---|
| RAM | 4 GB | Minimum comfortable for Server 2025 Desktop Experience |
| vCPUs | 2 | Minimum recommended for AD DS |
| Disk | 80 GB (`local-lvm`) | Headroom for OS, roles, and logs |
| BIOS | OVMF (UEFI) | Modern firmware, required for TPM |
| Machine | q35 | Required for UEFI/TPM support in Proxmox |
| Network | VirtIO, `vmbr0` bridged | Best performance, same physical network as clients |
| TPM | v2.0 | Required for Windows Server 2025 |

---

## VirtIO Driver Fix

The Windows installer couldn't see the disk. VirtIO SCSI drivers aren't bundled in the Windows ISO, so without them the disk is completely invisible during setup.

**Fix:**
- Downloaded `virtio-win.iso` from the Fedora People repository
- Attached it to `ide2` in the Proxmox VM hardware tab
- During Windows setup, used **Load Driver** and pointed it to `E:\amd64\2k22\viostor.inf`
- Disk showed up immediately and installation continued normally

This is a common Proxmox + Windows issue. Worth noting for any future Windows VMs built on this cluster.

---

## OS Installation

- Selected Desktop Experience for GUI-based administration
- Set local Administrator password
- Confirmed Server Manager loads on first boot

---

## Notes

UEFI and TPM were configured from the start because that's the current enterprise standard. Legacy BIOS isn't used here.

Bridged networking puts DC01 and the VirtualBox clients on the same `192.168.40.x` subnet as the physical network. Domain join works without any extra routing.

Static IP on DC01 is mandatory before AD DS promotion. DNS depends on a fixed address and so do the clients.

---

## Next Steps

1. Set hostname to `DC01`
2. Assign static IP on `192.168.40.x`
3. Configure DNS to point to itself
4. Snapshot - clean pre-promotion baseline
