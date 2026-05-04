# Phase 13 - Audit and Logging

## Overview

This phase configures Advanced Audit Policy on DC01 via GPO, enables object-level auditing on the file shares created in Phase 9, and validates that events are being generated correctly in the Windows Security event log. A reference list of key Event IDs is included at the end.

Audit policy configuration was deferred from Phase 10 to keep the work coherent. Phase 10 covered service hardening and firewall configuration. Audit policy is documented here as a standalone deliverable.

---

## Environment

| Setting | Value |
|---|---|
| Domain Controller | DC01 (`exodus.lab`) |
| OS | Windows Server 2025 |
| GPO Name | DC Security Audit Policy |
| GPO Linked To | Domain Controllers OU |
| File Shares Audited | `C:\Shares\IT`, `C:\Shares\HR`, `C:\Shares\Accounting`, `C:\Shares\Shared` |

---

## Why Advanced Audit Policy Instead of Basic Audit Policy

Windows has two audit policy mechanisms: the legacy basic audit policy under Security Settings, and the Advanced Audit Policy Configuration introduced in Windows Vista. They control the same underlying subcategories but at different levels of granularity.

The basic policy offers nine broad categories with no subcategory control. Advanced Audit Policy exposes 53 subcategories, allowing precise control over exactly what gets logged. On a domain controller this matters. Logging too little misses critical events, and logging too much generates millions of events per day and buries the signal in noise.

Advanced Audit Policy is the correct choice for any production or production-simulating environment.

One important caveat: Microsoft warns that mixing basic and advanced audit policy settings on the same machine can produce unpredictable results. To force Advanced Audit Policy to take precedence, the setting Audit: Force audit policy subcategory settings to override audit policy category settings should be enabled under Security Options. This was not required in this lab because basic audit policy settings were not configured, but it is worth noting for production deployments.

---

## Audit Policy Configuration

A dedicated GPO named **DC Security Audit Policy** was created and linked to the Domain Controllers OU. Keeping audit policy in its own GPO makes it easy to identify, modify, and audit the policy itself separately from other DC configuration.

### Before State

The following subcategories were already enabled from Windows Server 2025 defaults prior to GPO application, verified via `auditpol /get /category:*`:

<img width="762" height="708" alt="image" src="https://github.com/user-attachments/assets/a58320ef-0d8f-4981-8019-9649c695d5fa" />

<img width="576" height="718" alt="image" src="https://github.com/user-attachments/assets/84b14e01-408c-48de-9cef-849ce6b824d8" />



All other subcategories were set to No Auditing.

### GPO Configuration

The following subcategories were configured in the DC Security Audit Policy GPO under:
Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Account Logon | Audit Credential Validation | Success and Failure | Captures NTLM authentication attempts including failures that may indicate brute force |
| Account Logon | Audit Kerberos Authentication Service | Success and Failure | Captures Kerberos TGT requests and failures at the KDC |
| Account Logon | Audit Kerberos Service Ticket Operations | Success and Failure | Captures service ticket requests and failures, key for detecting Kerberoasting |
<img width="674" height="315" alt="image" src="https://github.com/user-attachments/assets/5925df66-ac5a-4a63-9d52-e40545925192" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Account Management | Audit Computer Account Management | Success and Failure | Captures computer account creates, modifies, and deletes |
| Account Management | Audit Security Group Management | Success and Failure | Captures changes to security groups including membership changes |
| Account Management | Audit User Account Management | Success and Failure | Captures user account creates, modifies, disables, and password resets |
<img width="728" height="400" alt="image" src="https://github.com/user-attachments/assets/94d32232-4a62-447a-8b07-da4ab2a5aab8" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Detailed Tracking | Audit Process Creation | Success | Captures all process launches on the DC, useful for detecting unusual execution |
<img width="658" height="407" alt="image" src="https://github.com/user-attachments/assets/5fdb9f39-531c-4b18-b5b8-51eb0fbc5588" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| DS Access | Audit Directory Service Access | Success and Failure | Captures reads and queries against AD objects |
| DS Access | Audit Directory Service Changes | Success and Failure | Captures creates, modifies, moves, and undeletes of AD objects |
<img width="776" height="376" alt="image" src="https://github.com/user-attachments/assets/463f7b10-ca43-4e66-bdd5-6c6bdb51d0a7" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Logon/Logoff | Audit Account Lockout | Success and Failure | Captures account lockout events |
| Logon/Logoff | Audit Logoff | Success | Captures session terminations |
| Logon/Logoff | Audit Logon | Success and Failure | Captures interactive and network logons and failures |
<img width="735" height="473" alt="image" src="https://github.com/user-attachments/assets/2a6b5cf6-70d0-45a9-89ab-46ff22845d53" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Object Access | Audit File Share | Success and Failure | Captures network share access events |
| Object Access | Audit File System | Success and Failure | Captures file and folder access events where SACLs are configured |
<img width="744" height="512" alt="image" src="https://github.com/user-attachments/assets/f53c4d6d-b26d-4946-8b16-3b5f8bbba8b7" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Policy Change | Audit Policy Change | Success and Failure | Captures changes to the audit policy itself |
| Policy Change | Audit Authentication Policy Change | Success and Failure | Captures changes to domain trust and authentication policies |
<img width="769" height="430" alt="image" src="https://github.com/user-attachments/assets/c167316f-66fc-43c8-8ab7-b078f9bb148b" />

---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| Privilege Use | Audit Sensitive Privilege Use | Success and Failure | Captures use of sensitive privileges such as SeDebugPrivilege and SeTakeOwnershipPrivilege |
<img width="793" height="400" alt="image" src="https://github.com/user-attachments/assets/cc068051-d90c-4269-b3e6-d143f0bdc0d2" />


---

| Category | Subcategory | Setting | Reason |
|---|---|---|---|
| System | Audit Security State Change | Success and Failure | Captures system startup, shutdown, and security subsystem state changes |
| System | Audit System Integrity | Success and Failure | Captures violations of security subsystem integrity |
<img width="782" height="452" alt="image" src="https://github.com/user-attachments/assets/da41f019-25f8-4d76-a021-a4a543ed6c72" />


---

**Subcategories intentionally left as No Auditing:**

| Subcategory | Reason |
|---|---|
| Non Sensitive Privilege Use | Extremely high volume on a DC, generates millions of events per day with low signal value |
| Directory Service Replication | No replication partners in this lab, not applicable |
| Detailed Directory Service Replication | Same as above |
| IPsec subcategories | Not in use in this environment |
| Filtering Platform subcategories | Not required for this lab scope |

### After State

Verified via `auditpol /get /category:*` after running `gpupdate /force`:

<img width="714" height="717" alt="image" src="https://github.com/user-attachments/assets/b560cf6b-b247-4e38-8cf5-a0d87564eebd" />
<img width="672" height="715" alt="image" src="https://github.com/user-attachments/assets/1d3bbbe3-d1ba-4018-9429-efb68914b55a" />



---

## File Share Object Auditing (SACLs)

The audit policy subcategories for File System and File Share control whether the audit infrastructure is active. They do not by themselves generate events for specific folders. Windows also requires a System Access Control List (SACL) on each folder specifying what access to audit and for whom.

SACLs were added to all four department shares created in Phase 9.

**SACL configuration applied to each folder:**

| Setting | Value |
|---|---|
| Principal | Everyone |
| Type | All (Success and Failure) |
| Applies to | This folder, subfolders and files |
| Permissions audited | Traverse folder / execute file, List folder / read data, Read attributes, Read extended attributes, Create files / write data, Create folders / append data, Write attributes, Write extended attributes, Delete, Read permissions, Change permissions |

The Everyone principal was used intentionally. In a production environment this would be scoped to specific groups to reduce event volume. For lab purposes, auditing all access demonstrates the full capability.

SACLs were configured via Properties > Security > Advanced > Auditing > Add on each folder.

---

## Validation

Three event types were triggered and confirmed in Event Viewer (Windows Logs > Security).

### Logon Event

Triggered by locking and unlocking the DC01 console session.

| Field | Value |
|---|---|
| Event ID | 4624 |
| Task Category | Logon |
| Keywords | Audit Success |
| Description | An account was successfully logged on |

### File System Access Event

Triggered by creating a test file at `C:\Shares\IT\audit-test.txt`.

| Field | Value |
|---|---|
| Event ID | 4663 |
| Task Category | File System |
| Keywords | Audit Success |
| Subject | EXODUS\ExodusAdmin |
| Description | An attempt was made to access an object |

### AD Object Change Event

Triggered by running `Set-ADUser -Identity "ExodusAdmin" -Description "DC01 Domain Admin"`.

| Field | Value |
|---|---|
| Event ID | 5136 |
| Task Category | Directory Service Changes |
| Keywords | Audit Success |
| Subject | EXODUS\ExodusAdmin |
| Description | A directory service object was modified |

Test artifacts were cleaned up after validation. `audit-test.txt` was removed from `C:\Shares\IT` and the Description attribute was cleared on ExodusAdmin.

---

## Key Event ID Reference

| Event ID | Category | Description | When It Fires |
|---|---|---|---|
| 4624 | Logon | Successful logon | Any successful interactive or network logon |
| 4625 | Logon | Failed logon | Any failed logon attempt, key for detecting brute force |
| 4634 | Logoff | Account logoff | Session terminated |
| 4648 | Logon | Logon using explicit credentials | RunAs or pass-the-hash indicator |
| 4663 | Object Access | Attempt to access object | File or folder accessed where SACL is configured |
| 4670 | Object Access | Permissions changed on object | NTFS permission modification |
| 4672 | Logon | Special privileges assigned | Admin-level logon |
| 4673 | Privilege Use | Sensitive privilege use attempted | Use of privileges like SeDebugPrivilege |
| 4688 | Process Creation | New process created | Any process launch on the DC |
| 4698 | Object Access | Scheduled task created | Persistence mechanism indicator |
| 4719 | Policy Change | Audit policy changed | Someone modified the audit policy |
| 4720 | Account Management | User account created | New AD user created |
| 4722 | Account Management | User account enabled | Account re-enabled |
| 4723 | Account Management | Password change attempt | User attempted to change their own password |
| 4724 | Account Management | Password reset attempt | Admin reset a user password |
| 4725 | Account Management | User account disabled | Account disabled |
| 4726 | Account Management | User account deleted | Account deleted from AD |
| 4728 | Account Management | Member added to security group | Group membership change |
| 4732 | Account Management | Member added to local group | Local group membership change |
| 4738 | Account Management | User account changed | Any attribute change on a user object |
| 4740 | Account Management | User account locked out | Account lockout threshold hit |
| 4743 | Account Management | Computer account deleted | Computer object removed from AD |
| 4756 | Account Management | Member added to universal security group | Universal group membership change |
| 4771 | Account Logon | Kerberos pre-authentication failed | Failed Kerberos logon, key for detecting password spraying |
| 4776 | Account Logon | NTLM authentication attempt | NTLM credential validation |
| 4768 | Account Logon | Kerberos TGT requested | Any Kerberos authentication attempt |
| 4769 | Account Logon | Kerberos service ticket requested | Service access via Kerberos, Kerberoasting generates unusual 4769s |
| 4946 | Policy Change | Firewall rule added | Windows Firewall rule change |
| 5136 | DS Access | Directory service object modified | Any AD object attribute changed |
| 5137 | DS Access | Directory service object created | New AD object created |
| 5139 | DS Access | Directory service object moved | AD object moved between OUs |
| 5141 | DS Access | Directory service object deleted | AD object deleted |

---

## Note on Centralised Log Collection

In a production environment, collecting events in the local Security event log on the DC is not sufficient. The event log has a finite size and rolls over, and there is no centralised visibility across multiple systems. The standard practice is to forward Windows Security events to a SIEM for centralised collection, alerting, correlation, and long-term retention. This lab does not have a SIEM deployed, so events are reviewed locally via Event Viewer only.

---

## Next Steps

This is the final phase of the lab as currently scoped. The environment is fully built and documented across all 13 phases.
