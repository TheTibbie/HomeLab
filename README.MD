# Homelab Documentation

## Overview

This repository documents the design, build-out, and ongoing operation of my homelab environment.

The lab started as a Proxmox-focused infrastructure project, but it has grown into a broader systems administration lab covering virtualization, network segmentation, firewall policy, backups, monitoring, self-hosted infrastructure services, and Active Directory.

The goal is not just to show that services are running. The goal is to document the architecture, implementation decisions, validation steps, issues encountered, and follow-up work behind the environment.

Sensitive details such as full IP addresses, MAC addresses, WAN details, credentials, serial numbers, and device-specific identifiers are intentionally redacted throughout.

---

## Current Status

The lab is operational and actively being expanded.

Current implemented areas include:

- 4-node Proxmox VE cluster
- Proxmox Backup Server design and maintenance documentation
- OPNsense active edge firewall and router
- VLAN segmentation across wired and wireless networks
- Managed switching through Omada
- VLAN-backed wireless networks
- Pi-hole DNS filtering
- Uptime Kuma monitoring
- Dashy service dashboard
- Active Directory lab with domain controller deployment and validation

Some items remain in cleanup, revalidation, or hardening phases. Those are documented directly instead of being hidden.

---

## Design Intent

The lab is built around production-style habits:

- Segment the network instead of leaving everything flat
- Use firewall policy to control inter-VLAN access
- Keep management access restricted and recoverable
- Use monitoring to validate service availability
- Document issues and recovery steps instead of only documenting the final state
- Keep backups, restore validation, and maintenance tasks visible
- Use virtualization boundaries instead of placing everything on one host
- Redact sensitive values so the documentation can be safely shared publicly

This repository is written as an infrastructure documentation portfolio, not a generic tutorial.

---

## Major Lab Areas

| Area | Description |
|---|---|
| Overview | Purpose, scope, architecture goals, and high-level design decisions |
| Proxmox | Cluster design, node layout, storage, backup planning, and operational notes |
| Networking | Pi-hole, OPNsense, VLAN segmentation, firewall policy, Omada, wireless, and cutover documentation |
| Active Directory | AD DS installation, domain promotion, OU structure, validation, and administrative workflow documentation |

---

## Current Architecture Summary

The environment is built around a Proxmox VE cluster hosting infrastructure and service workloads.

OPNsense now acts as the active routing and firewall layer. VLAN segmentation is implemented for workstation, IoT, guest, and management/server roles. Managed switching and wireless VLAN mapping are handled through Omada.

The current design includes a practical Management / Servers VLAN consolidation because some infrastructure still sits behind an unmanaged switch. A dedicated server VLAN remains staged for future use when the switching layer supports it cleanly.

Active Directory is also documented as a separate systems administration lab area, including domain controller setup, validation, and future administrative structure.

---

## Core Infrastructure

### Virtualization

- Proxmox VE cluster across multiple physical nodes
- VM and container-based workload separation
- Proxmox snapshots used during major configuration changes
- Cluster management and Corosync recovery documented after the VLAN cutover

### Network and Firewall

- OPNsense active router/firewall
- VLAN segmentation implemented
- Default-deny inter-VLAN posture where practical
- Explicit allow rules for required management and service paths
- DNS centralized through Pi-hole
- NTP centralized through OPNsense
- Omada-managed switch and wireless AP
- Temporary management/adoption path documented for switch/AP cleanup

### Monitoring and Dashboards

- Uptime Kuma used for service availability monitoring
- Dashy used as a convenience dashboard for service access
- Monitoring targets updated after network migration
- Service screenshots and URLs redacted before publishing

### Backups

- Proxmox Backup Server documented as part of the backup architecture
- Retention, garbage collection, and verification workflow documented
- Proxmox nodes validated against the PBS target on the new addressing scheme

### Active Directory

- Windows Server-based AD DS lab
- Domain controller installation and promotion documented
- DNS and domain validation documented
- OU structure, user/group lifecycle, and Group Policy documented
- Windows 11 client domain join validated
- File share permissions tested with role-based access
- Security hardening documented with before and after state
- PowerShell automation created for onboarding, offboarding, and reporting
- System State backup and restore procedure tested
- Advanced Audit Policy and key Event IDs documented

---

## Repository Layout

```text
README.MD
01-overview/
  Purpose, scope, architecture goals, and high-level documentation

02-proxmox/
  Proxmox cluster, nodes, storage, backups, maintenance, and operations

03-networking/
  Pi-hole, OPNsense, VLANs, firewall policy, Omada, wireless, and cutover documentation

04-active-directory/
  Active Directory Domain Services, domain controller configuration, OU structure, and administrative workflows
```

Folder numbering may evolve as the lab expands. The goal is clear documentation structure, not keeping the original numbering forever.

---

## Current Networking Milestone

The OPNsense and VLAN cutover is complete and documented.

The cutover documentation includes:

- Final topology and VLAN design
- Active OPNsense firewall and DHCP configuration
- Switch, wireless, and Omada configuration
- Proxmox and service migration
- Issues encountered and resolutions
- Post-cutover validation and cleanup

This section is important because it documents both the final architecture and the recovery work required during the cutover, including routing fixes, quorum recovery, switch recovery, DNS troubleshooting, and temporary management-path decisions.

---

## Current Active Directory Milestone

The Active Directory lab documents a complete Windows Server-based domain environment.

The documentation covers:

- VM creation and Windows Server installation
- Pre-promotion configuration
- AD DS role installation
- Domain controller promotion
- OU structure design
- User and group lifecycle management
- Group Policy configuration and validation
- DNS configuration and reverse lookup zones
- Windows 11 client setup and domain join
- File share permissions with role-based access testing
- Security hardening
- PowerShell automation
- System State backup and restore validation
- Audit policy and logging

This lab area demonstrates Windows Server administration, identity infrastructure, endpoint domain join, policy management, backup and recovery, automation, and audit logging alongside the Linux/networking portions of the homelab.

---

## What This Demonstrates

This repository demonstrates hands-on work across several infrastructure areas:

- Multi-node Proxmox cluster deployment and management
- Virtual machine and container workload separation
- OPNsense firewall and VLAN implementation
- Managed switch and wireless VLAN configuration
- DNS filtering and resolver design with Pi-hole
- Network cutover planning and recovery
- Proxmox quorum and Corosync troubleshooting
- Service monitoring with Uptime Kuma
- Dashboard organization with Dashy
- Backup planning and PBS maintenance documentation
- Active Directory Domain Services deployment, GPOs, automation, backup/recovery, and audit logging
- Operational documentation with screenshots, validation, and issue tracking
- Security-conscious redaction for public-facing documentation

---

## Documentation Standards

Each major section is written to document:

- What was built
- Why it was built that way
- What changed during implementation
- What issues came up
- How those issues were resolved
- What validation was performed
- What cleanup or hardening remains

The documentation intentionally includes imperfect parts of the build. Real infrastructure work includes misconfigurations, recovery steps, deferred decisions, and validation gaps. Those are documented directly so the repo reflects actual operational work instead of a polished but unrealistic final state.

---

## Roadmap

### Completed or Operational

- Proxmox cluster online
- Core services deployed
- Pi-hole DNS filtering online
- OPNsense deployed as active router/firewall
- VLAN segmentation implemented
- Omada-managed switch and wireless AP adopted
- Workstation, IoT, and guest wireless roles configured
- Uptime Kuma monitoring updated after migration
- Active Directory Domain Services lab fully built, tested, and documented

### Cleanup and Revalidation

- Continue periodic PBS backup and restore validation
- Review OPNsense aliases for stale staged VLAN references
- Remove temporary firewall or management paths when safe
- Review Dashy links after service migrations
- Confirm monitoring targets remain current
- Continue documenting deferred workloads as they become active

### Future Hardening

- Move switch and AP management fully onto the intended management model
- Reintroduce a dedicated server VLAN when the switching layer supports it cleanly
- Move selected workloads out of the Management / Servers VLAN where isolation provides a clear benefit
- Expand IoT and camera segmentation
- Continue backup and restore validation documentation
- Expand Active Directory administrative workflows and automation

---

## Public Documentation Note

This repository is designed to be safe to mirror publicly.

Internal implementation details are redacted where needed. Screenshots should be reviewed before publishing to remove:

- Full internal IP addresses
- MAC addresses
- WAN details
- Serial numbers
- UUIDs
- Credentials
- Identifiable client names
- Sensitive service URLs
- Any other environment-specific identifiers

The goal is to show the design and operational process without exposing unnecessary details.
