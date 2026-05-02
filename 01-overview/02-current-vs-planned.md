# Current State and Future Direction

## Current State

The homelab is operational and has moved beyond the original flat-network design. The environment now runs behind OPNsense with VLAN segmentation, managed switching, wireless VLAN mapping, DNS filtering, monitoring, backup documentation, and Active Directory lab documentation.

The purpose of this page is to summarize the current state of the lab and track the next areas of cleanup, validation, and hardening.

---

## Platform

- Proxmox VE cluster across `proxmox-01` through `proxmox-04`
- VM and container workloads distributed across the cluster
- Proxmox management moved to the Management / Servers VLAN after the OPNsense cutover
- Corosync ring addresses updated after the VLAN migration
- Guest placement, storage, and backup posture documented in the Proxmox section

---

## Networking and Edge

OPNsense is now the active router, firewall, DHCP service, and inter-VLAN control point for the environment.

Current networking state:

- OPNsense is active as the router and firewall
- VLAN segmentation is implemented across wired and wireless networks
- Omada-managed switching and wireless are in use
- Workstation, IoT, Guest, and Management / Servers roles are active
- DHCP is handled by OPNsense
- DNS is centralized through Pi-hole
- NTP is centralized through OPNsense
- Inter-VLAN access is controlled through firewall policy

The original server VLAN design was adjusted during implementation. Because part of the infrastructure still sits behind an unmanaged switch, management and server workloads are currently consolidated into the Management / Servers VLAN. The dedicated server VLAN remains staged for future use when the switching layer supports it cleanly.

---

## Services

Current documented services include:

- Pi-hole, dual instances for DNS filtering
- Uptime Kuma for availability monitoring
- Dashy as a service dashboard
- Home Assistant in a dedicated VM
- Media services hosted in a dedicated VM with bulk disk passthrough
- Active Directory lab hosted as a separate systems administration track

Service placement is documented where relevant. After the OPNsense cutover, several infrastructure services now sit on the Management / Servers VLAN.

---

## Backups

Proxmox Backup Server is documented as the centralized backup platform.

Current backup documentation covers:

- PBS storage configured for Proxmox
- Weekly Proxmox backup jobs
- Centralized retention on PBS
- Prune, garbage collection, and verify jobs
- Backup design decisions and maintenance posture

Any post-cutover PBS addressing or backup-path validation should remain tracked in the relevant Proxmox and OPNsense cutover documents.

---

## Current Cleanup and Revalidation

The major network cutover is complete, but cleanup and validation work remains.

Current cleanup items include:

- Remove the temporary Omada management/adoption path when safe
- Confirm switch and AP management fully through the intended management path
- Review OPNsense aliases for stale staged VLAN references
- Review temporary firewall rules created during the transition
- Confirm Pi-hole and media service aliases match active placement
- Confirm monitoring targets remain current after service moves
- Review Dashy links for stale service URLs
- Validate PBS addressing and backup paths after the VLAN cutover where needed

These are cleanup and hardening tasks. They are not blockers for the current operational state.

---

## Future Direction

The next phase is focused on refinement rather than initial deployment.

**Networking and Security**

- Fully retire temporary Omada management paths after validation
- Reintroduce a dedicated server VLAN when the switching layer supports it cleanly
- Move selected workloads out of the Management / Servers VLAN where isolation provides a clear benefit
- Continue tightening inter-VLAN policy and removing transitional rules
- Expand IoT and camera segmentation as more devices are added

**Operational Maturity**

- Continue backup and restore validation
- Keep monitoring targets aligned with service placement
- Document recovery procedures and lessons learned after major changes
- Continue improving screenshot redaction and public documentation hygiene

**Active Directory**

- Expand administrative workflows where useful
- Add more realistic client and user scenarios over time
- Continue documenting automation, auditing, backup, and recovery work
