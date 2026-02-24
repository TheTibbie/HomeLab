# Bulk Media Disk Passthrough (Host → Media VM)

## Intent

Keep the Proxmox host minimal — the media VM owns the bulk storage filesystem and all application-level layout decisions.

---

## Current implementation

- A large external disk on `proxmox-01` is labeled `media-drive` (ext4)
- The disk is passed through to VM 100 (`media-vm`) as a dedicated attached device
- Inside the VM, the disk is mounted and used for media and download paths

---

## Why this design

Passing the disk through to the VM rather than managing it at the hypervisor level keeps a clean boundary between infrastructure and application concerns. Storage layout, mount points, and filesystem decisions belong to the media VM — if that changes, the hypervisor layer doesn't need to be touched. It also scopes troubleshooting: a storage issue with the media stack starts and ends in the VM, not at the cluster level.
