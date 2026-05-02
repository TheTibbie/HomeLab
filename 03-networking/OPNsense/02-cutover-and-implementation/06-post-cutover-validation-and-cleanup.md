# Post-Cutover Validation and Cleanup

## Overview

This page documents the validation and cleanup work after the OPNsense VLAN cutover.

The cutover is operational. OPNsense is routing the environment, VLAN segmentation is active, the managed switch and wireless AP are adopted into Omada, Proxmox management access has been restored, and core services have been migrated or updated.

This page tracks what has been validated, what still needs cleanup, and what remains as future hardening work.

Full IP addresses, MAC addresses, WAN details, serial numbers, device-specific identifiers, and sensitive service URLs are intentionally redacted throughout.

---

## Validation Summary

| Area | Status | Notes |
|---|---|---|
| OPNsense routing | Validated | OPNsense is active as the router and firewall |
| VLAN segmentation | Validated | Workstation, IoT, Guest, and Management / Servers roles are active |
| DHCP | Validated | Kea DHCP is active for the current VLAN roles |
| DNS | Validated | Pi-hole DNS is active after firewall adjustment |
| NTP | Configured | Internal clients use OPNsense as the intended time source |
| Managed switch | Validated | Switch is adopted in Omada and carrying the required VLAN roles |
| Wireless AP | Validated | AP is adopted and broadcasting VLAN-backed wireless networks |
| Proxmox management | Validated | Nodes are reachable on the Management / Servers VLAN |
| Proxmox cluster quorum | Validated | Quorum was restored after Corosync recovery |
| Uptime Kuma | Validated | Monitoring targets were updated after service migration |
| Dashy | Operational | Dashboard is reachable, with links reviewed as services move |
| Media server / Jellyfin | Validated | Service is reachable after migration |
| Home Assistant | Operational | Current placement is Management / Servers |
| Proxmox Backup Server | Validated | PBS is online on the Management / Servers VLAN and backup paths were validated after the cutover |

---

## Network Validation

The network was validated after moving from the old flat network to the segmented VLAN design.

Validation confirmed:

- Workstation clients receive the expected VLAN placement
- Wireless clients can join the expected wireless networks
- Internet access works from trusted client networks
- Management services are reachable through the intended management path
- IoT and guest networks remain separate from internal infrastructure where policy applies
- TCP-based service checks work where firewall policy may block ICMP

Ping was not treated as the only validation method. Some firewall rules intentionally block or restrict ICMP, so TCP-based checks and actual service access were used where appropriate.

---

## OPNsense Validation

OPNsense was validated as the active routing and firewall layer.

Validation confirmed:

- OPNsense is the active router/firewall
- WAN routing is stable after removing the old LAN parent interface IP
- VLAN interfaces are active
- Kea DHCP is serving the active VLAN roles
- DNS enforcement is built around Pi-hole
- Pi-hole hosts can reach upstream DNS
- NTP is centralized through OPNsense
- Inter-VLAN policy is enforced through firewall rules

The old LAN parent interface issue was resolved during the cutover. The parent interface now acts as the VLAN trunk parent instead of carrying the old flat-network IP.

---

## Firewall Validation

Firewall validation focused on confirming that the network was functional without weakening segmentation.

Validated policy behavior includes:

| VLAN Role | Validation |
|---|---|
| Workstations | Trusted client access to required management and service ports |
| IoT | Restricted access with only required internal service paths allowed |
| Guest | Internet access without internal infrastructure access |
| Management / Servers | Infrastructure and migrated services reachable through required paths |
| Servers | Present as staged configuration, not active server placement |

DNS enforcement was also validated after Pi-hole moved into the Management / Servers VLAN. A firewall allow rule was added so Pi-hole could reach upstream DNS while clients still used Pi-hole as the resolver path.

---

## Switch and Wireless Validation

The managed switch and wireless AP were validated after the switch recovery and Omada adoption work.

Validation confirmed:

- Managed switch is adopted into Omada
- Switch port roles are restored
- OPNsense trunk connectivity is working
- Workstation access port is working
- Infrastructure uplink is working
- AP trunk is working
- Recovery port remains available
- Wireless AP is adopted into Omada
- Workstation and IoT wireless networks were tested with live clients
- Guest wireless is present as a separate role

A temporary Omada management/adoption path remains in place. This is a controlled transitional state and should not be removed until the replacement management path is confirmed.

---

## Proxmox Validation

Proxmox validation focused on restoring cluster management after moving nodes to the Management / Servers VLAN.

Validation confirmed:

- Proxmox nodes are reachable after the VLAN move
- Proxmox management access was restored
- Corosync was updated to use the new management placement
- Cluster quorum returned after Corosync recovery
- The OPNsense host remains reachable while still carrying OPNsense WAN and LAN/trunk roles

The initial cutover caused quorum loss because Corosync still referenced the old flat-network addresses. That was resolved by updating the Corosync ring addresses, bumping the configuration version, distributing the corrected configuration, and restarting Corosync.

---

## Service Validation

Core services were validated or updated after the cutover.

| Service | Validation Status |
|---|---|
| Omada Controller | Reachable from trusted management path |
| Pi-hole DNS | Operational after firewall adjustment |
| Uptime Kuma | Updated and monitoring migrated services |
| Dashy | Reachable after migration |
| Media server / Jellyfin | Reachable after migration |
| Home Assistant | Reachable in current placement |
| Proxmox Backup Server | Online on the Management / Servers VLAN with backup paths validated |

Service validation focused on actual reachability instead of only checking whether a VM or container was powered on.

---

## Monitoring Validation

Uptime Kuma was updated after the migration because service addresses changed during the cutover.

Validation confirmed:

- Migrated services were added or updated
- Stale monitor targets were corrected where identified
- Monitoring reflects the current network design
- Uptime Kuma is the primary service-health dashboard

Dashy was also moved and reviewed, but it is treated as a convenience dashboard rather than the source of truth for service health.

---

## Current Cleanup Items

The network is operational, but cleanup remains.

Tracked cleanup items include:

- Remove the temporary Omada management/adoption path when safe
- Confirm switch and AP management fully through the intended management VLAN
- Review OPNsense aliases for stale server VLAN references
- Review firewall rules created during the transition
- Remove temporary rules that are no longer required
- Confirm Pi-hole aliases match the active placement
- Confirm media server aliases match the active placement
- Review Uptime Kuma monitors for stale targets
- Review Dashy links for stale service URLs
- Continue periodic PBS backup and restore validation

These items are cleanup and hardening work, not blockers for the current operational state.

---

## PBS Post-Cutover Validation

Proxmox Backup Server was already implemented and tested before the VLAN cutover.

After the cutover, PBS was migrated onto the Management / Servers VLAN and validated on the new addressing scheme. Proxmox nodes can reach PBS, the PBS storage target remains available in Proxmox, and backup paths have been validated after the network migration.

Post-cutover PBS validation confirmed:

- PBS has the correct Management / Servers VLAN placement
- Proxmox nodes can reach PBS on the new network
- Proxmox storage configuration points to the correct PBS target
- Backup jobs still target the expected datastore
- Maintenance jobs remain scheduled
- Backup paths survived the VLAN migration

PBS should still be included in periodic operational validation. Backup systems can appear healthy until a restore or scheduled job exposes a pathing or permissions issue, so ongoing backup and restore checks remain part of the hardening roadmap.

---

## Hardening Roadmap

Future hardening work includes:

- Fully move switch and AP management onto the intended management VLAN
- Remove the temporary/default Omada management path
- Reintroduce the dedicated server VLAN when switching supports it cleanly
- Move selected workloads out of the Management / Servers VLAN where isolation provides a clear benefit
- Tighten broad transitional firewall rules
- Expand IoT and camera segmentation as more devices are added
- Continue validating monitoring after service moves
- Continue documenting periodic backup and restore testing

These are design maturity tasks. They are not required to restore basic network functionality.

---

## Status

The post-cutover state is operational.

OPNsense is routing the environment, VLAN segmentation is active, wireless networks are mapped to VLANs, Proxmox management access has been restored, core services have been migrated or updated, and monitoring reflects the current service layout.

Remaining work is focused on cleanup, periodic backup validation, temporary management path removal, and future hardening.
