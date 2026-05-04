# Phase 5 - User & Group Lifecycle Management

## Overview

With the OU structure in place, user accounts and security groups were created following a consistent naming convention. Onboarding and offboarding procedures are documented here along with PowerShell equivalents for everything done through the GUI.

---

## Naming Conventions

| Object | Format | Example |
|---|---|---|
| Username | firstinitial.lastname | jsmith |
| Security Group | GRP_ prefix | GRP_IT |
| Distribution List | DL_ prefix | DL_AllStaff |

AD is case-insensitive for authentication and lookups. SamAccountNames work identically regardless of capitalization.

---

## Users Created

| Full Name | Username | Department OU |
|---|---|---|
| James Smith | JSmith | IT |
| Emily Carter | ECarter | HR |
| Robert Miles | RMiles | Accounting |
| Sarah Connor | SConnor | Admins |

All accounts were created with an initial password and User must change password at next logon enforced.

<img width="374" height="182" alt="image" src="https://github.com/user-attachments/assets/a6f6445c-bd56-4a4e-a9cd-6b0233477714" /><img width="374" height="182" alt="image" src="https://github.com/user-attachments/assets/a41a4d41-6d37-48cd-adbe-1f3f73c90781" /><img width="374" height="182" alt="image" src="https://github.com/user-attachments/assets/cec440ac-0cb1-4163-a0ab-10676b221510" /><img width="374" height="182" alt="image" src="https://github.com/user-attachments/assets/6a93e84b-c8a6-4c9a-bcbf-d608d6ca85ed" />

<img width="961" height="720" alt="image" src="https://github.com/user-attachments/assets/c0167f5f-e18e-4e83-bd19-d42251e5ea5a" />






---

## Onboarding Procedure

1. In ADUC, right-click the correct department OU, select New, then User
2. Fill in first name, last name, and logon name (`firstinitial.lastname`)
3. Set initial password with must change at next logon checked
4. Add user to their department security group

PowerShell equivalent:

```powershell
New-ADUser -Name "First Last" `
           -GivenName "First" `
           -Surname "Last" `
           -SamAccountName "flast" `
           -UserPrincipalName "flast@Exodus.lab" `
           -Path "OU=Department,OU=Users,OU=Exodus,DC=Exodus,DC=lab" `
           -AccountPassword (ConvertTo-SecureString "Welcome1!" -AsPlainText -Force) `
           -ChangePasswordAtLogon $true `
           -Enabled $true

Add-ADGroupMember -Identity "GRP_Department" -Members "flast"
```

---

## Security Groups Created

| Group Name | Scope | Type | OU |
|---|---|---|---|
| GRP_IT | Global | Security | Groups\Security |
| GRP_HR | Global | Security | Groups\Security |
| GRP_Accounting | Global | Security | Groups\Security |
| GRP_Admins | Global | Security | Groups\Security |

<img width="498" height="330" alt="image" src="https://github.com/user-attachments/assets/c7e4a83c-c16f-4e62-a817-229299fe32f4" />

---

## Distribution Groups Created

| Group Name | Scope | Type | OU | Members |
|---|---|---|---|---|
| DL_AllStaff | Global | Distribution | Groups\Distribution | All four users |

Distribution groups are used for email distribution. Functional testing requires an Exchange or M365 environment which is out of scope for this lab. The group was created to demonstrate naming convention and structure.

<img width="572" height="346" alt="image" src="https://github.com/user-attachments/assets/745ba7b0-18c7-4ee0-bf85-78b4d5a666d2" />

---

## Offboarding Procedure

Three steps, completed in order:

**Step 1 - Disable the account:**

```powershell
Disable-ADAccount -Identity flast
```

**Step 2 - Strip all group memberships:**

```powershell
Remove-ADGroupMember -Identity "GRP_Department" -Members "flast" -Confirm:$false
```

**Step 3 - Move to Disabled OU:**

```powershell
Move-ADObject -Identity "CN=First Last,OU=Department,OU=Users,OU=Exodus,DC=Exodus,DC=lab" `
              -TargetPath "OU=Disabled,OU=Exodus,DC=Exodus,DC=lab"
```

Accounts are retained in the Disabled OU for a minimum of 6 months before permanent deletion.

---

## Verified With

```powershell
# Verify user placement
Get-ADUser -Filter * -SearchBase "OU=Users,OU=Exodus,DC=Exodus,DC=lab" | Select-Object Name, DistinguishedName

# Verify group membership
Get-ADGroupMember -Identity "GRP_IT"

# Verify distribution group membership
Get-ADGroupMember -Identity "DL_AllStaff" | Select-Object Name, SamAccountName

# Verify offboarding
Get-ADUser -Identity rmiles -Properties MemberOf, Enabled | Select-Object Name, Enabled, DistinguishedName, MemberOf
```

---

## Next Steps

1. Configure Default Domain Policy password settings
2. Build and link workstation lockdown GPO
3. Test and validate GPO application on domain-joined client
