# Active Directory Enterprise Lab

## Summary

This documents the build-out of a Windows Server 2025 Active Directory environment hosted on Proxmox and VirtualBox. The goal is a realistic enterprise setup, not just a working domain. That means proper role separation, a structured OU hierarchy, tested GPOs, PowerShell automation, and a backup process that has actually been validated.

The lab was built and documented phase by phase as each piece was completed.

> **Status: Complete** - All 13 phases built, tested, and documented.

---

## Network Context

This Active Directory lab was originally built on the pre-cutover flat lab network. The individual phase documents preserve the addressing and validation state from when the lab was built.

That is intentional. The AD documentation represents the implementation state at the time of the AD build. The later OPNsense VLAN cutover is documented separately under:

`../03-networking/OPNsense/02-cutover-and-implementation/`

If the AD lab is migrated into the post-cutover VLAN design later, that migration should be documented as a separate update rather than rewriting the original build history.

---

## Design Intent

**AD DS and DNS only on DC01.** DHCP was left off DC01 intentionally. In production environments, DHCP typically lives on network infrastructure, not domain controllers. The lab is built the same way.

**Proxmox for the DC, VirtualBox for clients.** DC01 runs on `proxmox-03` in the existing cluster. The Windows 11 clients run under VirtualBox on the main Windows PC. The lab was originally built with the DC and clients on the same internal lab subnet through bridged networking, so domain join worked without extra routing.

**UEFI + TPM from the start.** DC01 uses OVMF firmware, q35 machine type, and TPM v2.0. Legacy BIOS is not used here.

**GUI first, PowerShell second.** Everything gets configured through the GUI first so there is a clear understanding of what each step does. PowerShell equivalents are documented alongside as a record of what was built and how it could be scripted.

**Security is a planned phase, not a retrofit.** Hardening, audit policy, and logging are scoped into the roadmap from the start. CIS Benchmark baselines, advanced audit policy via GPO, and Event ID documentation are all tracked.

---

## Environment

| Component | Detail |
|---|---|
| Domain | `exodus.lab` |
| DC Hypervisor | Proxmox (`proxmox-03`) |
| Client Hypervisor | VirtualBox (main Windows PC) |
| DC OS | Windows Server 2025 Standard (Desktop Experience) |
| Client OS | Windows 11 |
| Network | Internal lab subnet, originally built before the OPNsense VLAN cutover |
| DC Roles | AD DS, DNS only |

### VM Inventory

| VM | Host | RAM | Role |
|---|---|---|---|
| `DC01` | `proxmox-03` | 4 GB | AD DS, DNS |
| `EXO-WKS-001` | VirtualBox | 2-3 GB | Windows 11 domain workstation |
| `EXO-WKS-002` | VirtualBox | 2-3 GB | Windows 11 domain workstation |

---

## Roadmap

### Phase 1 - VM Creation & OS Installation *(complete)*

- VM provisioned on `proxmox-03` with correct firmware and driver stack
- VirtIO SCSI driver issue resolved
- Windows Server 2025 Desktop Experience installed and confirmed booting

### Phase 2 - Pre-Promotion Configuration *(complete)*

- Hostname set to `DC01`
- Static IP assigned on the internal lab subnet
- DNS configured to point to itself
- Pre-promotion snapshot taken

### Phase 3 - AD DS Installation & Domain Promotion *(complete)*

- AD DS role installed via Server Manager
- Promoted to domain controller, new forest `exodus.lab` created
- DNS zones created automatically and validated
- Post-promotion snapshot taken

### Phase 4 - OU Structure Design *(complete)*

- OU hierarchy designed and built to reflect enterprise role and resource separation
- All OUs created via GUI with PowerShell equivalents documented
- Protect from Accidental Deletion enabled on all OUs

### Phase 5 - User & Group Lifecycle Management *(complete)*

- Standard users and security groups created with consistent naming convention
- Onboarding and offboarding procedures documented with PowerShell equivalents
- Distribution group created for email structure demonstration

### Phase 6 - Group Policy Objects *(complete)*

- Password and lockout policy configured in Default Domain Policy
- Four GPOs built and linked to Workstations OU: lockdown, software restriction, admin restrictions, local admin enforcement
- GPO precedence documented and application validated on domain-joined client

### Phase 7 - DNS Configuration *(complete)*

- Reverse lookup zone created for the original internal lab subnet
- Static PTR record added for DC01
- Forward and reverse resolution validated from DC01

### Phase 8 - Windows 11 Client Setup & Domain Join *(complete)*

- `EXO-WKS-001` and `EXO-WKS-002` provisioned in VirtualBox and domain joined
- Both computers moved to Workstations OU post-join
- GPO application validated via `gpresult`, domain user login confirmed

### Phase 9 - File Share Permissions *(complete)*

- Four department shares created on DC01 under `C:\Shares\`
- Share permissions set broadly, NTFS permissions handle granular access control
- Role-based access tested and validated with file creation tests

### Phase 10 - Security Hardening *(complete)*

- Built-in Administrator account renamed
- Print Spooler and WinRM disabled with orphaned firewall rules cleaned up
- DefaultInboundAction set to Block on all firewall profiles
- CIS Benchmark-aligned controls applied: NTLMv2 only, anonymous SAM enumeration blocked
- All changes documented with before and after state

### Phase 11 - PowerShell Automation *(complete)*

- Four scripts built: bulk user import, single user onboarding, offboarding, and interactive AD reporting
- All scripts handle failure states, log actions, and prompt before making changes
- Scripts stored at `C:\Scripts\AD\` on DC01 and committed to the repository

### Phase 12 - Backup & Recovery *(complete)*

- Dedicated BACKUP01 VM provisioned as backup target
- Windows Server Backup installed and System State backup completed to `\\BACKUP01\BackupShare`
- Full restore procedure tested and validated in DSRM
- Runbook documented in markdown

### Phase 13 - Audit & Logging *(complete)*

- Advanced Audit Policy configured via dedicated GPO linked to Domain Controllers OU
- SACLs added to all four department shares for file system auditing
- Logon, file access, and AD object change events validated in Event Viewer
- Key Event ID reference documented

---

## Repository Layout

```text
04-active-directory/
├── README.md
├── 01-installation.md
├── 02-pre-promotion.md
├── 03-domain-promotion.md
├── 04-ou-structure.md
├── 05-user-group-lifecycle.md
├── 06-group-policy.md
├── 07-dns-configuration.md
├── 08-client-setup.md
├── 09-file-share-permissions.md
├── 10-security-hardening.md
├── 11-powershell-automation.md
├── 12-backup-recovery.md
└── 13-audit-logging.md
```

---

## What This Demonstrates

- **Active Directory deployment from scratch:** forest creation, DNS integration, and domain controller promotion on modern hardware with UEFI/TPM
- **Enterprise OU design:** hierarchy built around real role and resource separation
- **Group Policy architecture:** multiple GPOs with defined scope, correct OU linkage, and validated application
- **Identity lifecycle management:** onboarding and offboarding procedures with group membership and OU discipline
- **DNS administration:** forward/reverse zones, static records, and end-to-end resolution validation
- **File share security:** NTFS and share permission separation with least-privilege design and tested role-based access
- **Security hardening:** CIS Benchmark-aligned changes with documented before/after state
- **PowerShell automation:** scripted lifecycle management and AD reporting, documented and version controlled
- **Backup and recovery:** System State backups with a tested restore procedure and a written runbook
- **Audit and logging:** Advanced Audit Policy via GPO with key Event IDs identified and validated
