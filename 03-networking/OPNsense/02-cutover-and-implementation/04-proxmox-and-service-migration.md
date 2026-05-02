# Proxmox and Service Migration

## Overview

This page documents the Proxmox and service migration work completed during the OPNsense VLAN cutover. The main goal was moving Proxmox management, supporting infrastructure, and core services from the old flat network onto the new segmented network without losing cluster access.

Full IP addresses, MAC addresses, device-specific identifiers, and sensitive service URLs are intentionally redacted throughout.

---

## Migration Goal

The cutover required moving infrastructure services onto the new Management / Servers VLAN while preserving access to Proxmox nodes, Proxmox Backup Server, OPNsense, Omada Controller, Pi-hole DNS, monitoring, dashboards, and migrated service workloads.

Because several Proxmox nodes and supporting systems remained connected through an unmanaged switch, those systems were consolidated into the Management / Servers VLAN for this phase. This kept the cutover stable without requiring additional cabling or switching redesign mid-migration.

---

## Proxmox Management Migration

The Proxmox nodes were moved from the old flat network to the Management / Servers VLAN.

| System Group | VLAN Placement |
|---|---|
| Proxmox nodes | Management / Servers |
| Proxmox Backup Server | Management / Servers |
| Management interfaces | Management / Servers |
| Infrastructure services | Management / Servers |
| Current server workloads | Management / Servers |

The migration was handled carefully because losing Proxmox access would also affect recovery options for OPNsense, service containers, and virtual machines.

<img width="219" height="165" alt="Proxmox node status after management migration" src="https://github.com/user-attachments/assets/049c6efb-5734-4641-9a90-6663ecc8668c" />

---

## DHCP-First Cutover Approach

The initial migration used DHCP before confirming final static management addresses. This reduced the risk of losing access mid-cutover. If a static address was applied incorrectly, a node could become unreachable. DHCP allowed nodes to come up automatically on the new VLAN and made them findable through OPNsense leases.

After nodes were reachable on the new network, static addressing and service-level configuration were cleaned up safely.

---

## Proxmox Cluster and Corosync

After the nodes moved to the new Management / Servers VLAN, Corosync was still referencing old flat-network addresses. The Corosync configuration had to be updated to match the new management placement.

The update included:

- Replacing old Corosync ring addresses with new management VLAN addresses
- Bumping the Corosync configuration version
- Distributing the corrected configuration to all nodes
- Restarting Corosync so the cluster could re-form

A temporary quorum issue made the normal cluster filesystem path unavailable for editing. The corrected configuration had to be staged and copied directly during recovery. After the update, the cluster regained quorum and Proxmox management access was restored.

---

## OPNsense Host Consideration

The Proxmox host running the OPNsense VM required special care. Because this host provides the firewall and routing layer, changes to its network configuration could affect the entire environment.

The OPNsense host uses separate connectivity for:

- Proxmox management
- OPNsense WAN
- OPNsense LAN/trunk

The management interface was adjusted so the Proxmox host remained reachable from the Management / Servers VLAN while OPNsense continued carrying VLAN traffic toward the managed switch. This avoided mixing Proxmox management, WAN, and VLAN trunking into a single unclear network path.

---

## Proxmox Backup Server

Proxmox Backup Server has been migrated onto the Management / Servers VLAN.

Post-cutover validation confirmed:

- Proxmox nodes can reach Proxmox Backup Server on the new addressing scheme
- PBS storage remains available in Proxmox
- Existing backup jobs still target the expected datastore
- Backup paths have been validated after the OPNsense VLAN cutover
- PBS maintenance posture remains documented through the Proxmox backup section

This confirms PBS is online after the network migration and is no longer only a pre-cutover backup component.

---

## Service Migration Summary

| Service | Current Placement | Migration Status |
|---|---|---|
| Omada Controller | Management / Servers | Migrated and operational |
| Pi-hole DNS instances | Management / Servers | Migrated and operational |
| Uptime Kuma | Management / Servers | Migrated and monitoring services |
| Dashy | Management / Servers | Migrated and reachable |
| Media server / Jellyfin | Management / Servers | Migrated and reachable |
| Home Assistant | Management / Servers | Migrated and reachable |
| Proxmox Backup Server | Management / Servers | Migrated, online, and validated after the cutover |

<img width="923" height="861" alt="Service inventory after migration to Management / Servers VLAN" src="https://github.com/user-attachments/assets/12cc0963-444a-49cd-8d07-4f8699ae84d1" />

---

## Omada Controller Migration

Omada was moved onto the Management / Servers VLAN early in the cutover because it manages the switch and AP. Losing controller access during the cutover complicated switch recovery, so restoring it became a priority. After migration, Omada was reachable from the trusted workstation network through the required management path.

<img width="887" height="781" alt="Omada Controller reachable after service migration" src="https://github.com/user-attachments/assets/fb7a3cb2-ba98-46ed-ba66-f336e94d3de2" />

---

## Pi-hole Migration

The Pi-hole instances were moved into the Management / Servers VLAN. The original staged design placed them in the dedicated server VLAN, which was deferred during implementation. After migration, OPNsense DHCP was updated to hand out the Pi-hole instances as DNS servers to client VLANs, and a firewall rule was added to allow Pi-hole to reach upstream DNS resolvers.

DNS and firewall behavior is documented in `02-opnsense-firewall-and-dhcp.md`.

---

## Uptime Kuma Migration

Uptime Kuma was updated after the service migration because multiple monitored services moved from the old flat network to the new Management / Servers VLAN. Monitoring targets were updated so the dashboard reflected the current network state. Uptime Kuma is now reporting migrated services successfully.

---

## Dashy Migration

Dashy was moved onto the Management / Servers VLAN with the other migrated services. After the network migration, dashboard links should be reviewed so they continue pointing to current service locations.

Dashy is treated as a convenience layer, not the source of truth for service health. Uptime Kuma remains the better validation point for monitoring whether services are reachable.

<img width="957" height="813" alt="Dashy dashboard after migration to Management / Servers VLAN" src="https://github.com/user-attachments/assets/6718fe9f-b972-41fd-a2a9-da87360b7031" />

---

## Media Server / Jellyfin Migration

The media server was moved onto the new addressing scheme and validated after the cutover. Jellyfin access was restored from trusted client networks. Because media access may be needed from IoT devices such as smart TVs, firewall policy allows only the required media service path rather than broad IoT access to internal infrastructure.

---

## Home Assistant Migration

Home Assistant is currently placed in the Management / Servers VLAN. Future placement can be reviewed as IoT segmentation matures since it may require controlled communication with IoT devices.

---

## Deferred Workloads

Some workloads were intentionally deferred from the main cutover scope. These will be handled separately once the core network has stabilized. Deferral kept the cutover focused on restoring and validating the critical environment first.

---

## Service Validation

Validation confirmed:

- Proxmox nodes reachable on the Management / Servers VLAN
- Proxmox cluster quorum restored
- Proxmox Backup Server reachable from Proxmox on the new addressing scheme
- PBS storage target available in Proxmox
- Omada Controller reachable from trusted clients
- Pi-hole DNS operational after firewall adjustment
- Uptime Kuma updated and reporting migrated services
- Dashy reachable after migration
- Jellyfin reachable after media server migration
- Home Assistant reachable in the new placement

TCP-based checks were used where firewall policy could block ICMP.

---

## Follow-Up Items

- Confirm all Proxmox nodes remain reachable on the Management / Servers VLAN
- Continue periodic Proxmox Backup Server backup and restore validation
- Review Uptime Kuma monitors for stale service targets
- Review Dashy links for stale service URLs
- Review Home Assistant placement after IoT segmentation matures
- Move deferred workloads only after the core network remains stable
- Decide whether selected workloads should move to a dedicated server VLAN in the future

---

## Status

Proxmox management access has been restored on the Management / Servers VLAN. Corosync was updated for the new management placement, the cluster regained quorum, and core infrastructure services were migrated or updated.

Proxmox Backup Server is online on the Management / Servers VLAN, reachable from the Proxmox nodes, and validated as part of the post-cutover backup path.

The current service placement is operational. Remaining work is focused on cleanup, periodic validation, and deferred workload planning.
