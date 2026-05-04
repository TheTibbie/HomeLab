# Phase 10 - Security Hardening

## Overview

Baseline security hardening applied to DC01 after domain promotion. The goal is to reduce the attack surface of the domain controller by disabling unnecessary services, cleaning up orphaned firewall rules, enforcing an explicit inbound deny posture, and applying CIS Benchmark-aligned authentication controls.

This is not a comprehensive hardening guide. It covers the settings most relevant to a lab DC running Windows Server 2025 with no printing infrastructure, no remote management requirement, and no legacy client authentication needs.

---

## Environment

| Setting | Value |
|---|---|
| Hostname | DC01 |
| Domain | `exodus.lab` |
| NetBIOS | EXODUS |
| OS | Windows Server 2025 |
| Admin Account | ExodusAdmin (renamed from Administrator) |

---

## 1. Administrator Account Rename

The built-in Administrator account was renamed to `ExodusAdmin` via Local Security Policy.

**Path:** `secpol.msc` > Local Policies > Security Options > Accounts: Rename administrator account

Renaming the built-in account removes a known, fixed attack target. Brute-force and credential stuffing tools routinely target `Administrator` by name. It doesn't prevent compromise on its own, but it eliminates a guaranteed username from the attacker's starting position.

<img width="910" height="265" alt="Screenshot 2026-04-16 094148" src="https://github.com/user-attachments/assets/58d60639-6747-4ffd-bf78-c75e77c05b57" />


> **Note:** The rename takes effect on the next login. PowerShell sessions opened before re-login will still show `exodus\administrator` in `whoami` output. This is expected and resolves after a fresh login as `ExodusAdmin`.


---

## 2. Unnecessary Services Disabled

Two services were identified as unnecessary on a domain controller and disabled. Neither has a legitimate function on DC01 and both increase attack surface without providing any operational value.

### Print Spooler

DC01 has no printers attached and will never route print jobs. The Print Spooler service is also the vector for PrintNightmare (CVE-2021-1675 and CVE-2021-34527), a critical privilege escalation vulnerability that specifically affects the Spooler service on domain controllers.

**GUI Steps:**

1. Open `services.msc`
2. Right-click Print Spooler > Properties
3. Click Stop
4. Set Startup type to Disabled
5. Click Apply > OK

**PowerShell Equivalent:**

```powershell
Stop-Service -Name Spooler
Set-Service -Name Spooler -StartupType Disabled
```

**State Change:**

| Property | Before | After |
|---|---|---|
| Startup Type | Automatic | Disabled |
| Status | Running | Stopped |
<img width="399" height="463" alt="image" src="https://github.com/user-attachments/assets/b12fb264-107b-46ab-a8d5-6197b712bc5b" />


---

### Windows Remote Management (WinRM)

DC01 is managed exclusively from the VM console via Proxmox. No remote PowerShell sessions are used. Leaving WinRM running exposes an unnecessary remote management interface on the domain controller.

**GUI Steps:**

1. Open `services.msc`
2. Right-click Windows Remote Management (WS-Management) > Properties
3. Click Stop
4. Set Startup type to Disabled
5. Click Apply > OK

**PowerShell Equivalent:**

```powershell
Stop-Service -Name WinRM
Set-Service -Name WinRM -StartupType Disabled
```

**State Change:**

| Property | Before | After |
|---|---|---|
| Startup Type | Automatic | Disabled |
| Status | Running | Stopped |
<img width="405" height="467" alt="image" src="https://github.com/user-attachments/assets/d5664723-1445-45c3-8db2-405d5b132fde" />


---

### Combined Verification

```powershell
Get-Service -Name Spooler, WinRM | Select-Object Name, Status, StartType
```
<img width="929" height="139" alt="image" src="https://github.com/user-attachments/assets/22b0c3f8-e49f-4ef0-811b-bc79b76f64d5" />





---

## 3. Windows Defender Firewall

### Profile State

All three firewall profiles were already enabled before this phase started. No change required.

```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled
```

| Profile | Enabled |
|---|---|
| Domain | True |
| Private | True |
| Public | True |

### Orphaned Rule Cleanup

Disabling Print Spooler and WinRM leaves their inbound firewall rules active with no service behind them. Rules for disabled services were cleaned up.

**WinRM Inbound Rules:**

```powershell
Get-NetFirewallRule -DisplayName "Windows Remote Management (HTTP-In)" | Disable-NetFirewallRule

Get-NetFirewallRule -DisplayName "Windows Remote Management (HTTP-In)" | Select-Object DisplayName, Enabled, Profile
```

| Rule | Before | After |
|---|---|---|
| Windows Remote Management (HTTP-In) - Domain, Private | Enabled | Disabled |
| Windows Remote Management (HTTP-In) - Public | Enabled | Disabled |

<img width="1260" height="170" alt="image" src="https://github.com/user-attachments/assets/16804f0e-a6b9-4296-87e4-77b0906071ce" />

**Print Spooler Inbound Rules:**

```powershell
Get-NetFirewallRule | Where-Object { $_.DisplayName -like "*Spooler*" } | Disable-NetFirewallRule

Get-NetFirewallRule | Where-Object { $_.DisplayName -like "*Spooler*" } | Select-Object DisplayName, Enabled, Profile
```

All Spooler inbound rules set to Disabled across all profiles.

<img width="1255" height="346" alt="image" src="https://github.com/user-attachments/assets/1a55d2f8-9221-4b96-847c-8c91beed75b2" />


### Default Inbound Action

Windows Firewall defaults to `NotConfigured` for `DefaultInboundAction`, meaning inbound traffic with no matching rule is not explicitly blocked by policy. Setting this to `Block` makes the deny posture explicit.

Before applying this, existing enabled inbound rules were reviewed to confirm all AD-required traffic is covered:

- Kerberos Key Distribution Center (TCP/UDP)
- Active Directory Domain Controller - LDAP (TCP/UDP), Secure LDAP
- Active Directory Domain Controller - SAM/LSA, RPC, RPC-EPMAP, W32Time, Echo Request
- DNS (TCP/UDP, Incoming)
- DFS Replication, File Replication (RPC, RPC-EPMAP)
- Core Networking rules

```powershell
Set-NetFirewallProfile -All -DefaultInboundAction Block

Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

| Profile | Before | After |
|---|---|---|
| Domain | NotConfigured | Block |
| Private | NotConfigured | Block |
| Public | NotConfigured | Block |

<img width="1223" height="166" alt="image" src="https://github.com/user-attachments/assets/85d86ce2-4510-45ea-bedc-a9fb79f49c0c" />


> **Note:** Outbound remains `NotConfigured` intentionally. Locking down outbound on a DC without a validated outbound ruleset risks breaking AD replication, DNS forwarding, NTP synchronisation, and Kerberos ticket issuance. Outbound hardening is out of scope for this phase.

---

## 4. CIS Benchmark Baseline

Settings drawn from the CIS Microsoft Windows Server 2025 Benchmark. All changes applied via `secpol.msc` and verified against the registry.

### LAN Manager Authentication Level - NTLMv2 Only

**Path:** `secpol.msc` > Local Policies > Security Options > Network security: LAN Manager authentication level

Without this set explicitly, Windows negotiates the highest mutually supported authentication version, which means a client can force NTLMv1 or LM and succeed. Setting this to `Send NTLMv2 response only. Refuse LM & NTLM` forces the DC to reject anything that isn't NTLMv2.

**GUI Steps:**

1. Open `secpol.msc`
2. Navigate to Local Policies > Security Options
3. Double-click Network security: LAN Manager authentication level
4. Select Send NTLMv2 response only. Refuse LM & NTLM
5. Click Apply > OK

<img width="1183" height="198" alt="image" src="https://github.com/user-attachments/assets/06837f58-a2f8-4148-9965-5344f875aa63" />


**PowerShell Equivalent:**

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "LmCompatibilityLevel" -Value 5
```

**State Change:**

| Setting | Before | After |
|---|---|---|
| LAN Manager authentication level | Not Defined | Send NTLMv2 response only. Refuse LM & NTLM |
| Registry: LmCompatibilityLevel | Not set | 5 |

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "LmCompatibilityLevel"
```

<img width="1210" height="199" alt="image" src="https://github.com/user-attachments/assets/7848462a-d0f4-41dd-8546-101642e57eb8" />



---

### Anonymous Enumeration of SAM Accounts and Shares

**Path:** `secpol.msc` > Local Policies > Security Options > Network access section

Anonymous enumeration allows unauthenticated connections to pull account names, groups, and share names from a system. On a DC this includes domain account information. Both policies below should be enabled.

| Policy | Description |
|---|---|
| Do not allow anonymous enumeration of SAM accounts | Blocks unauthenticated enumeration of account names |
| Do not allow anonymous enumeration of SAM accounts and shares | Extends the block to include share enumeration |

**GUI Steps:**

1. Open `secpol.msc`
2. Navigate to Local Policies > Security Options
3. Double-click Network access: Do not allow anonymous enumeration of SAM accounts and shares
4. Set to Enabled
5. Click Apply > OK

> Do not allow anonymous enumeration of SAM accounts was already Enabled. No change required.

<img width="985" height="122" alt="image" src="https://github.com/user-attachments/assets/d3a20db7-e283-424f-ac56-0f269ef0600c" />


**PowerShell Equivalent:**

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymousSAM" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymous" -Value 1
```

**State Change:**

| Policy | Before | After |
|---|---|---|
| Do not allow anonymous enumeration of SAM accounts | Enabled | Enabled (no change) |
| Do not allow anonymous enumeration of SAM accounts and shares | Disabled | Enabled |
| Registry: RestrictAnonymousSAM | 1 | 1 (no change) |
| Registry: RestrictAnonymous | 0 | 1 |

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymousSAM", "RestrictAnonymous"
```

<img width="1260" height="241" alt="image" src="https://github.com/user-attachments/assets/e23fdf1a-95af-4923-b690-0ee5c75ffa36" />


---

## 5. Audit Policy

Audit policy configuration is deferred to Phase 13 (Audit & Logging) to keep that work coherent and complete rather than split across phases.

---

## Change Summary

| Category | Change | Method |
|---|---|---|
| Services | Print Spooler disabled | services.msc + PowerShell |
| Services | WinRM disabled | services.msc + PowerShell |
| Firewall | WinRM inbound rules disabled | PowerShell |
| Firewall | Spooler inbound rules disabled | PowerShell |
| Firewall | DefaultInboundAction set to Block on all profiles | PowerShell |
| CIS | LAN Manager auth level set to NTLMv2 only | secpol.msc + registry verified |
| CIS | Anonymous SAM enumeration blocked | secpol.msc + registry verified |
| CIS | Anonymous SAM + shares enumeration blocked | secpol.msc + registry verified |

---

## Next Steps

1. Build onboarding, offboarding, and AD reporting scripts in PowerShell
2. Comment and add all scripts to the repository
