# Phase 6 - Group Policy Objects

## Overview

Four GPOs were created and linked to the Workstations OU, plus the Default Domain Policy was updated with a proper password and lockout configuration. Each GPO covers a specific area of workstation control. Settings were chosen to reflect realistic enterprise policy without breaking legitimate functionality.

---

## Password & Lockout Policy (Default Domain Policy)

Configured via the Default Domain Policy, applies domain-wide.

| Setting | Before | After |
|---|---|---|
| Minimum Password Length | 7 characters | 12 characters |
| Account Lockout Threshold | 0 (disabled) | 5 invalid attempts |
| Lockout Duration | Not defined | 10 minutes |
| Reset Lockout Counter | Not defined | 10 minutes |
| Complexity Enabled | True | True (unchanged) |
| Password History | 24 | 24 (unchanged) |
| Max Password Age | 42 days | 42 days (unchanged) |

```powershell
Get-ADDefaultDomainPasswordPolicy
```
<img width="773" height="372" alt="image" src="https://github.com/user-attachments/assets/1e710b9b-65f0-4c2a-8ed3-0e9266fc4269" />
<img width="560" height="242" alt="image" src="https://github.com/user-attachments/assets/2134eb15-5e56-4d4e-978c-4ba2108faec3" />


---

## GPOs Linked to Workstations OU

All four GPOs below are linked to `OU=Workstations,OU=Computers,OU=Exodus,DC=Exodus,DC=lab`.

---

### GPO 1 - Workstation Lockdown Policy

**Screensaver / Screen Lock**

| Setting | Path | Value |
|---|---|---|
| Enable screen saver | User Config > Admin Templates > Control Panel > Personalization | Enabled |
| Screen saver timeout | User Config > Admin Templates > Control Panel > Personalization | 900 seconds (15 min) |
| Password protect screen saver | User Config > Admin Templates > Control Panel > Personalization | Enabled |

Screensaver and password protection together function as a screen lock. After 15 minutes of inactivity, users must re-authenticate.

<img width="1129" height="450" alt="image" src="https://github.com/user-attachments/assets/03af81a5-bde3-4fd1-a5a0-715e8137b546" />

---


**USB / Removable Storage Restrictions**

| Setting | Path | Value |
|---|---|---|
| Removable Disks: Deny read access | Computer Config > Admin Templates > System > Removable Storage Access | Enabled |
| Removable Disks: Deny write access | Computer Config > Admin Templates > System > Removable Storage Access | Enabled |

<img width="1126" height="466" alt="image" src="https://github.com/user-attachments/assets/6cf9bb8e-1af5-4c9a-a565-67fbbcc54d8e" />


---

### GPO 2 - Software Restriction Policy

Default security level left as Unrestricted to avoid breaking legitimate software. Two targeted path rules added to block execution from common malware staging locations.

| Rule | Path | Security Level |
|---|---|---|
| Block AppData execution | `%AppData%\*.exe` | Disallowed |
| Block Temp execution | `%Temp%\*.exe` | Disallowed |


Setting the default to Disallowed without a full application whitelist risks breaking Windows components, admin tools, and user applications. Targeted rules are the safer and more practical enterprise approach.

<img width="1250" height="514" alt="image" src="https://github.com/user-attachments/assets/89f9366f-38d6-4574-b86e-9d447cdd61b9" />


---

### GPO 3 - Admin Restrictions Policy

Standard users are restricted from tools commonly used to bypass policy or make unauthorized system changes. Control Panel was intentionally left accessible since UAC handles privilege elevation natively for settings that require admin rights.

| Setting | Path | Value |
|---|---|---|
| Prevent access to command prompt | User Config > Admin Templates > System | Enabled (batch scripts also disabled) |
| Prevent access to registry editing tools | User Config > Admin Templates > System | Enabled (silent regedit also disabled) |
| Block PowerShell | User Config > Admin Templates > System > Don't run specified Windows applications | `powershell.exe`, `powershell_ise.exe` |

<img width="1075" height="470" alt="image" src="https://github.com/user-attachments/assets/02654613-737c-496e-9844-84f3d453d218" />


---

### GPO 4 - Local Administrator Restriction Policy

Configured via Restricted Groups to enforce that only Domain Admins are members of the local Administrators group on workstations. Any domain user previously holding local admin rights is removed when policy applies.

| Setting | Path | Value |
|---|---|---|
| Restricted Group | Computer Config > Policies > Windows Settings > Security Settings > Restricted Groups | Administrators, Members: Domain Admins only |

<img width="892" height="405" alt="image" src="https://github.com/user-attachments/assets/19385948-c875-4a6f-8b94-23e65d8515ae" />

---

## GPO Link Verification

```powershell
# Verify all GPOs exist
Get-GPO -All | Select-Object DisplayName, GpoStatus

# Verify GPOs linked to Workstations OU
Get-GPInheritance -Target "OU=Workstations,OU=Computers,OU=Exodus,DC=Exodus,DC=lab" | Select-Object -ExpandProperty GpoLinks
```
<img width="773" height="225" alt="image" src="https://github.com/user-attachments/assets/53551fe1-0ce2-49b1-b3cd-b18e70694f71" />
<img width="1267" height="612" alt="image" src="https://github.com/user-attachments/assets/b3469062-9d3b-4878-931c-7a752c332e88" />


---

## GPO Precedence (Link Order)

| Order | GPO |
|---|---|
| 1 | Workstation Lockdown Policy |
| 2 | Software Restriction Policy |
| 3 | Admin Restrictions Policy |
| 4 | Local Administrator Restriction Policy |

Lower link order number wins in the event of a conflict.

---

## Next Steps

1. Validate forward and reverse lookup zones for `exodus.lab`
2. Add static A records for lab infrastructure
3. Test DNS resolution from DC and from domain-joined client
