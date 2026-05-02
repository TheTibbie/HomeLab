# Standards and Next Steps

## Naming Conventions

- Nodes follow a sequential naming pattern: `proxmox-01` through `proxmox-04`
- Guests use short, descriptive names, usually service first and role second
- Examples include `pihole`, `pihole-b`, `media-vm`, `uptime-kuma`, and `dashy`

Consistent naming makes inventory, monitoring, backups, and troubleshooting easier to follow across the cluster.

---

## Redaction Standards

This section follows the repo-wide redaction policy documented in [Public repo redaction policy](../../01-overview/03-redaction-policy.md).

In brief:

- Internal IPs are redacted or generalized
- Tokens, keys, join secrets, VPN configs, PBS fingerprints, client inventories, and identifying logs are not published
- Screenshots are cropped or blurred if they show IPs, WAN details, usernames, hostnames, exposed ports, or other sensitive identifiers

---

## Current Baseline

The Proxmox cluster is operational and now runs as part of the post-cutover VLAN design.

Current baseline:

- Proxmox management is on the Management / Servers VLAN
- OPNsense is the active router and firewall for the environment
- DHCP is handled by OPNsense
- Cluster nodes use `vmbr0` as the primary Linux bridge for VM and container connectivity
- Corosync `ring0` addresses were updated to the Management / Servers VLAN after the OPNsense cutover
- Proxmox Backup Server is documented as the centralized backup platform
- Supporting infrastructure guests such as Pi-hole, Uptime Kuma, and Dashy are documented in their relevant sections

The old flat LAN was the original pre-cutover design. It is retained in historical preparation notes where relevant, but it is no longer the current networking baseline.

---

## Cluster Standards

Current cluster standards:

- Keep node naming consistent
- Keep guest names short and role-based
- Use Proxmox guests for service separation instead of placing all services on one host
- Use PBS as the centralized backup target
- Keep retention and maintenance tasks centralized on PBS where possible
- Document major changes before and after implementation
- Use snapshots before high-risk infrastructure changes when appropriate
- Validate cluster quorum after network or Corosync changes
- Keep management access restricted to trusted paths

---

## Operational Notes

The OPNsense VLAN cutover required updating Corosync ring addresses after Proxmox nodes moved from the old flat network to the Management / Servers VLAN.

That recovery is documented in:

`../../03-networking/OPNsense/02-cutover-and-implementation/05-issues-encountered-and-resolutions.md`

The key lesson is that Proxmox cluster networking should be treated as a dependency during major network changes. If management addressing changes, Corosync and quorum behavior must be validated before assuming the cluster is healthy.

---

## Next Steps

The next phase is focused on cleanup, validation, and hardening rather than initial deployment.

**Cluster and Platform**

- Continue validating Proxmox cluster health after network changes
- Keep node and guest inventory current
- Review guest placement as workloads grow
- Confirm PBS addressing and backup paths after VLAN changes where needed
- Continue documenting restore validation and operational recovery steps

**Networking and Access**

- Remove temporary Omada management paths only after safe validation
- Keep Proxmox management access restricted to trusted networks
- Review whether any Proxmox-related services should move out of the Management / Servers VLAN later
- Revisit dedicated server VLAN placement when switching supports it cleanly

**Operational Maturity**

- Continue backup and restore testing
- Keep monitoring targets current
- Maintain change notes for major infrastructure changes
- Review documentation periodically for stale network assumptions
