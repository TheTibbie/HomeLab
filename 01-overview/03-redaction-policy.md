# Public Repo Redaction Policy

## Goal

Share architecture, implementation, and operational work publicly without exposing sensitive details or creating avoidable attack surface.

The goal is to show the design and operational process clearly while keeping environment-specific values protected.

---

## What Gets Redacted

**Network identifiers**

- Full internal IP addresses
- WAN IPs and upstream provider details
- Public-facing addresses
- Open port listings
- NAT or port-forwarding details
- Firewall rule details that expose unnecessary internal structure

Internal addressing may be generalized when needed, for example:

```text
10.x.x.x
192.168.x.x
Management / Servers VLAN address
Workstation VLAN address
```

**Credentials and secrets**

- Passwords
- Tokens
- API keys
- Join secrets
- VPN configuration files
- WireGuard or OpenVPN keys
- Recovery keys
- Certificates or private keys
- Proxmox Backup Server fingerprints if they identify the environment
- Datastore paths with identifying details

**Device and identity details**

- MAC addresses
- Serial numbers
- Device UUIDs
- Identifiable client names
- Personal usernames
- Full client inventories
- Hostnames that expose private naming patterns
- SSID names if they reveal personal or private information

**Service details**

- Full internal URLs
- Service URLs that include hostnames or IP addresses
- Management interface addresses
- Public exposure details
- Sensitive dashboard views
- Alerting targets or notification endpoints

---

## Screenshot Hygiene

Screenshots are reviewed before publishing.

Before a screenshot is added to the public repo, it should be checked for:

- Full internal IP addresses
- WAN details
- MAC addresses
- Serial numbers
- UUIDs
- Usernames
- Hostnames with identifying information
- SSID names if they are sensitive
- Service URLs
- Public exposure details
- Logs containing private values
- Client names or device inventories

Screenshots should be cropped to show only what supports the point being documented. If a screenshot requires so much redaction that it becomes hard to understand, a text summary should be used instead.

---

## Documentation Redaction Standard

The repo should preserve enough technical detail to demonstrate the work without publishing unnecessary identifiers.

Good examples:

```text
OPNsense is the active router and firewall.
Pi-hole is distributed through OPNsense DHCP.
The Proxmox nodes are on the Management / Servers VLAN.
The switch and AP are adopted into Omada.
```

Avoid publishing:

```text
Exact WAN IPs
Exact management URLs
Full firewall exposure details
Raw VPN configuration files
Full DHCP lease/client tables
Unredacted screenshots of device inventories
```

---

## Security Posture

The environment is built to minimize unnecessary exposure.

Current posture includes:

- OPNsense as the active router and firewall
- VLAN segmentation across wired and wireless networks
- Inter-VLAN access controlled by firewall policy
- DNS centralized through Pi-hole
- DHCP handled by OPNsense
- Management services restricted to trusted paths
- Public documentation redacted before publishing

Remote access and management paths should not be documented in a way that exposes unnecessary attack surface.

---

## Public Repo Rule

If a detail helps explain the architecture, implementation decision, validation step, or operational process, it can usually be included in a generalized or redacted form.

If a detail only helps identify, target, or fingerprint the environment, it should be removed or redacted.
