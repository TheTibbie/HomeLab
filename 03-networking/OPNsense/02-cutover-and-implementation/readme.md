# OPNsense VLAN Cutover and Implementation

## Overview

This section documents the implementation of OPNsense and VLAN segmentation in the homelab. The previous section (`01-preparation`) covers the staged configuration built before the cutover. This section covers what was actually implemented, what changed during the cutover, what issues came up, and how the environment was stabilized.

The goal is to show the full operational process behind the migration, not just the final design.

Sensitive details including full IP addresses, MAC addresses, WAN details, and device-specific identifiers are intentionally redacted throughout.

---

## What Was Built

This cutover moved the homelab from a flat network to a segmented VLAN design with OPNsense handling routing, DHCP, firewall policy, and inter-VLAN control. The migration introduced managed switching and centrally managed wireless so that wired ports and SSIDs could be assigned to the correct VLANs.

At the end of the cutover:

- OPNsense is running as the active router and firewall
- VLAN segmentation is active across wired and wireless
- Managed switch is adopted into Omada
- Wireless AP is adopted into Omada with SSIDs mapped to workstation, IoT, and guest networks
- Proxmox management access is restored on the management VLAN
- Core services have been migrated onto the new addressing scheme
- Uptime Kuma is updated and monitoring migrated services

---

## Design Change During Cutover

The original staged design placed server workloads into a dedicated server VLAN. During implementation that design was adjusted.

Several Proxmox nodes and supporting infrastructure remained connected behind an unmanaged switch. Because that switch cannot pass tagged VLAN traffic, devices behind it had to share a single untagged VLAN. To preserve connectivity and reduce cutover risk, management and server workloads were consolidated into the management VLAN for this phase.

The dedicated server VLAN remains present in the configuration but is not the active server placement model. This is an implementation decision made to match available hardware, not a gap in the original design.

---

## VLAN Roles

| VLAN | Purpose |
|---|---|
| Workstations | Primary trusted client devices |
| IoT | Smart TVs, wireless test clients, and future camera infrastructure |
| Guest | Guest wireless clients with restricted internal access |
| Management / Servers | Proxmox nodes, management interfaces, monitoring, controller services, and current server workloads |
| Servers | Present in config, staged for future use when hardware supports full VLAN tagging |

---

## Physical Topology

```
ISP / Upstream Network
        |
        v
OPNsense VM on Proxmox host
        |
        v
Managed Switch
        |
        +-- Workstation access port
        +-- Wireless AP trunk
        +-- Proxmox / infrastructure uplink
        +-- Recovery / emergency access port
```

A full topology diagram with VLAN tagging detail is in `01-final-topology-and-vlan-design.md`.

---

## Firewall and Security Posture

- Default-deny between VLANs
- Explicit allow rules only where needed
- Management access restricted to trusted devices and required ports
- IoT and guest networks isolated from internal infrastructure
- DNS centralized through Pi-hole
- NTP centralized through OPNsense
- Temporary rules and transitional paths documented clearly

---

## Supporting Documentation

| File | Contents |
|---|---|
| `01-final-topology-and-vlan-design.md` | Full topology diagram, VLAN assignments, and IP scheme |
| `02-opnsense-firewall-and-dhcp.md` | Active OPNsense interface config, DHCP, firewall rules, and aliases |
| `03-switch-wireless-and-omada.md` | Switch port layout, VLAN config, AP SSIDs, and Omada adoption |
| `04-proxmox-and-service-migration.md` | Proxmox management VLAN migration and service migration status |
| `05-issues-encountered-and-resolutions.md` | Major issues encountered during the cutover and how they were resolved |
| `06-post-cutover-validation-and-cleanup.md` | Validation checks and post-cutover hardening items |

---

## Post-Cutover Hardening

The following items remain tracked as intentional follow-up work:

- Remove or retire the temporary VLAN 1 Omada adoption path once safe
- Confirm switch and AP can be fully managed from the intended management VLAN
- Clean up unused staged VLAN references in OPNsense
- Review firewall aliases for stale server VLAN entries
- Confirm Pi-hole and media server aliases match active placement
- Decide whether the dedicated server VLAN will be reintroduced when hardware supports it
- Continue moving deferred workloads as needed

---

## Status

The cutover is operational. OPNsense is routing the environment, VLAN segmentation is active, wireless networks are mapped to VLANs, and core services have been migrated or updated.

Remaining work is focused on cleanup, validation, and future hardening rather than restoring basic functionality.
