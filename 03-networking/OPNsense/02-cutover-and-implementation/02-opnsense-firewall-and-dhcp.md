# OPNsense Firewall and DHCP

## Overview

This page documents the active OPNsense configuration after the VLAN cutover. OPNsense now handles routing, DHCP, firewall policy, DNS enforcement, NTP access, and inter-VLAN control for the homelab. The original staged configuration was adjusted during the cutover because server workloads were consolidated into the Management / Servers VLAN for this phase.

Full IP addresses, MAC addresses, WAN details, and device-specific identifiers are intentionally redacted throughout.

---

## Active OPNsense Role

OPNsense is the active router and firewall for the environment. Current responsibilities include:

- Routing between VLANs
- Acting as the default gateway for each active VLAN
- Providing DHCP through Kea DHCP
- Enforcing inter-VLAN firewall policy
- Controlling DNS egress
- Providing NTP access to internal clients
- Providing outbound NAT through the WAN interface

---

## Interface Role Summary

| Interface Role | Purpose |
|---|---|
| WAN | Upstream connection toward the ISP/modem |
| LAN parent / trunk | Parent interface carrying VLAN traffic toward the managed switch |
| Workstations VLAN | Trusted client devices and administrative workstation access |
| IoT VLAN | Smart TVs, wireless test clients, and future IoT/camera devices |
| Guest VLAN | Guest wireless clients with restricted internal access |
| Management / Servers VLAN | Proxmox nodes, management interfaces, monitoring, controller services, DNS, and current server workloads |
| Servers VLAN | Present from the staged design, not currently the active server placement model |

The LAN parent interface is the VLAN trunk parent only. It does not carry an IP address from the old flat network.

<img width="874" height="507" alt="OPNsense interface assignments after the VLAN cutover" src="https://github.com/user-attachments/assets/3be57412-c00b-40c2-b76f-9e6e78eb0d6d" />

---

## Interface Fix During Cutover

During the cutover, the LAN parent interface still had an old staging IP from the previous flat network. This put both the WAN and LAN parent interfaces in the same upstream network range, which broke the default route. Outbound traffic was leaving through the wrong interface.

The fix was removing the old IP from the LAN parent interface and leaving it as the trunk parent only. After that change, OPNsense picked up the correct default route through WAN and internet access stabilized.

---

## DHCP Design

Kea DHCP handles all DHCP services. Scopes are configured per VLAN so clients receive the correct IP range, default gateway, DNS servers, and network options.

During staging, Kea was configured but left disabled to avoid conflicts with the old flat-network DHCP. It was enabled during the cutover as part of bringing the segmented network online.

**DHCP Scope Summary:**

| VLAN Role | DHCP Status | Notes |
|---|---|---|
| Workstations | Active | Trusted wired and wireless clients |
| IoT | Active | IoT wireless and future IoT/camera devices |
| Guest | Active | Guest wireless clients |
| Management / Servers | Active | Management, infrastructure, and migrated services |
| Servers | Staged | Not currently the active server placement model |

<img width="870" height="549" alt="OPNsense Kea DHCP scopes after the VLAN cutover" src="https://github.com/user-attachments/assets/e3c8e9cb-dc5e-45be-bc23-69652cfe9c10" />

---

## DNS Design

DNS is centralized through Pi-hole. The original staged design placed Pi-hole in the server VLAN. During implementation, Pi-hole was moved into the Management / Servers VLAN because the dedicated server VLAN was deferred.

OPNsense DHCP provides the Pi-hole instances as DNS servers to all client VLANs.

Intended DNS policy:

- Clients use Pi-hole for DNS
- Direct DNS to external resolvers is blocked where enforcement is enabled
- Pi-hole is allowed to reach upstream DNS
- DNS visibility stays centralized rather than allowing each VLAN to use arbitrary external resolvers

**Pi-hole firewall fix during cutover.** After Pi-hole moved to the Management / Servers VLAN, the firewall initially blocked Pi-hole from reaching its upstream DNS resolvers. The fix was an explicit allow rule permitting the Pi-hole hosts to reach upstream DNS on the required ports. This preserved DNS enforcement for clients while keeping Pi-hole functional as the resolver.

---

## NTP Design

NTP is centralized through OPNsense. Internal clients use OPNsense for NTP, OPNsense syncs upstream, and VLANs do not need broad outbound NTP access. This keeps time synchronization predictable and makes log correlation easier across OPNsense, Proxmox, monitoring tools, and self-hosted services.

---

## Firewall Policy Model

The firewall follows a default-deny approach between VLANs. Only explicitly required paths are allowed.

| VLAN | Policy |
|---|---|
| Workstations | Trusted client access to management and service ports, DNS, NTP, internet |
| IoT | DNS, NTP, internet, and specific allowed internal services only |
| Guest | DNS, NTP, and internet only, no internal access |
| Management / Servers | Restricted to required administrative and service paths |
| Servers | Staged, no active policy required yet |

---

## Workstations Policy

The workstation VLAN is the primary trusted client network. During implementation, workstation rules were updated so trusted clients could reach required services that had moved into the Management / Servers VLAN.

Allowed:

- Management access to infrastructure services
- Service access to migrated workloads
- DNS to Pi-hole
- NTP to OPNsense
- Internet access

Blocked:

- Direct DNS to external resolvers
- Unnecessary access to untrusted VLANs

<img width="872" height="574" alt="OPNsense workstation VLAN firewall rules" src="https://github.com/user-attachments/assets/8a0a15f1-cd58-4604-9925-a5afa45a06bd" />

---

## IoT Policy

IoT devices are restricted to what they need to function.

Allowed:

- DNS to Pi-hole
- NTP to OPNsense
- Internet access required for device functionality
- Specific internal service access where needed such as media service access

Blocked:

- Access to management interfaces
- Broad access to internal infrastructure
- Lateral movement into trusted VLANs

---

## Guest Policy

The guest VLAN has no internal access.

Allowed:

- DNS to Pi-hole
- NTP to OPNsense
- Internet access

Blocked:

- Access to management interfaces
- Access to server workloads
- Access to Proxmox infrastructure
- Access to other internal VLANs

---

## Management / Servers Policy

This VLAN currently carries both management infrastructure and migrated server workloads. Access is not treated as open even though trusted services live here. It is still restricted to required management and service paths.

Current workloads in this VLAN:

- Proxmox nodes and Proxmox Backup Server
- Omada Controller
- Pi-hole DNS instances
- Uptime Kuma and Dashy
- Media server / Jellyfin
- Home Assistant

Long term, some workloads may be moved out of this VLAN when the switching layer supports it and isolation provides a clear benefit.

<img width="857" height="514" alt="OPNsense Management / Servers VLAN firewall rules" src="https://github.com/user-attachments/assets/c53ac434-9c7b-4651-81eb-e35df9a30f79" />

---

## Server VLAN Status

The server VLAN is present in OPNsense but is not the active server placement model. It is staged for future use when the physical topology supports clean VLAN separation for all infrastructure hosts.

---

## Firewall Aliases

Aliases are used throughout the firewall rules to keep policy readable and maintainable. Updating an alias is safer than editing individual rules when service locations change during migration.

| Alias Type | Purpose |
|---|---|
| Host aliases | Key internal systems such as DNS instances, media server, and management targets |
| Port aliases | Common service ports such as DNS, NTP, web, and media services |
| Network aliases | VLAN groupings used for internal, untrusted, or restricted policy logic |

**Alias cleanup items:**

- Confirm Pi-hole aliases match the active Management / Servers placement
- Confirm media server alias matches the active placement
- Review internal VLAN group aliases for stale server VLAN references
- Remove unused staged references if the dedicated server VLAN remains deferred
- Confirm temporary transition rules are still required before keeping them

<img width="968" height="1232" alt="OPNsense firewall aliases used for VLAN policy" src="https://github.com/user-attachments/assets/45b21a5f-1a52-45a0-8175-fcb2391da6c1" />

---

## Key Firewall Changes During Cutover

- Added workstation access rules for services that moved into the Management / Servers VLAN
- Added allow rule for Pi-hole instances to reach upstream DNS
- Re-enabled DNS enforcement after Pi-hole became functional on the new network
- Removed or bypassed rules that depended on the dedicated server VLAN as the active placement model
- Kept temporary transitional paths in place and documented until they can be safely removed

---

## Follow-Up Items

- Complete alias cleanup and verify Pi-hole and media server aliases match active placement
- Review and remove any temporary firewall rules that are no longer needed
- Review staged server VLAN references and remove them if the dedicated server VLAN remains deferred

---

## Status

OPNsense is operational as the active router and firewall. DHCP is active through Kea, DNS enforcement is centered around Pi-hole, NTP is centralized through OPNsense, and inter-VLAN access is controlled by firewall policy.

Remaining work is focused on alias cleanup, staged VLAN cleanup, and reviewing temporary rules left in place during the cutover.
