# Shared ISO Library via NFS (PBS-hosted)

## What this provides

A shared ISO library hosted on PBS and mounted by all cluster nodes. Upload an ISO once, use it from any node — no per-node transfers or duplicate copies.

---

## Why this design

Hosting ISOs on PBS over NFS keeps image storage central without making it critical infrastructure. ISOs are re-downloadable, so the share is intentionally treated as disposable — loss is inconvenient, not destructive. The NFS approach also works independently of cluster state, which was useful during initial build-out when not all nodes were online simultaneously.

---

## Current configuration

| Detail | Value |
|---|---|
| NFS host | `192.168.x.x` (PBS server) |
| Export path | `/srv/nfs/isos` |
| Proxmox storage ID | `iso-nfs` |
| Content types | ISO images |

---

## Security posture

NFS is scoped to the LAN and restricted to known nodes — it's not exposed beyond the local network. The share is intentionally low-stakes: no credentials, no sensitive data, and no operational dependency if it's unavailable. Exact node IPs and NFS export rules are not published in this repo.
