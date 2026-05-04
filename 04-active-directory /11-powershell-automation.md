# Phase 11 - PowerShell Automation

## Overview

This phase covers four PowerShell scripts written to automate common Active Directory administrative tasks on DC01. The goal is to replace manual, error-prone processes for user onboarding, offboarding, and reporting with consistent, logged, and repeatable scripts that reflect how these tasks are handled in a real environment.

All scripts are stored on DC01 at `C:\Scripts\AD\` and can be run from an elevated PowerShell session or by right-clicking and selecting Run with PowerShell.

---

## Environment

| Setting | Value |
|---|---|
| Domain | `exodus.lab` |
| NetBIOS | EXODUS |
| Base User OU | `OU=Users,OU=Exodus,DC=Exodus,DC=lab` |
| Disabled OU | `OU=Disabled,OU=Exodus,DC=Exodus,DC=lab` |
| Script Location | `C:\Scripts\AD\` |
| SAMAccountName Format | `flastname` (first initial + lowercase last name) |
| Group Naming Convention | `GRP_Department` |

---

## Scripts

| File | Purpose |
|---|---|
| `New-BulkUsers.ps1` | Create multiple users from a CSV file |
| `New-OnboardUser.ps1` | Onboard a single new user interactively |
| `Remove-OffboardUser.ps1` | Offboard a user, disable account, strip groups, move to Disabled OU |
| `Get-ADReports.ps1` | Interactive menu-driven reporting and account management tool |
| `users.csv` | Sample CSV input file for bulk import |

<img width="750" height="471" alt="image" src="https://github.com/user-attachments/assets/3795a3e2-3425-429b-b520-7ef94b68f2db" />

---

## New-BulkUsers.ps1

Reads a CSV file and provisions AD accounts in bulk. Each row is processed independently so a failed row is logged and skipped without stopping the rest of the import. Every action is written to `C:\Scripts\AD\bulk-user-import.log`.

**Decision flow:**

1. Validate the CSV exists at the specified path, exit if not found
2. Validate all required columns are present, exit if any are missing
3. For each row, check that no required fields are blank and that the department is valid, skip row if not
4. Check if the SAMAccountName already exists in AD, skip row if it does
5. Create the account object in the correct department OU, disabled with no password set
6. Set the password via `Set-ADAccountPassword` and validate against domain policy
7. If the password fails, remove the orphaned account object, log the failure, move to the next row
8. If the password passes, enable the account and add the user to `GRP_Department` and `DL_AllStaff`
9. Log the result for each row and print a summary at the end showing created, skipped, and failed counts



<img width="1029" height="611" alt="image" src="https://github.com/user-attachments/assets/14f17a5b-b514-43f9-b469-74a8d26ff3e7" />


---

## New-OnboardUser.ps1

Creates a single user account from named parameters. Intended for individual onboarding requests where a full CSV import isn't needed. Prints a verification summary on completion.

**Decision flow:**

1. Check if the SAMAccountName already exists in AD, exit if it does
2. Validate the department against the list of valid values, exit if invalid
3. Create the account object in the correct department OU, disabled with no password set
4. Set the password via `Set-ADAccountPassword` and validate against domain policy
5. If the password fails, remove the orphaned account object and exit with an error
6. If the password passes, enable the account and add the user to `GRP_Department` and `DL_AllStaff`
7. Print a verification summary showing the DN, enabled state, and group memberships

<img width="790" height="551" alt="image" src="https://github.com/user-attachments/assets/e5d47604-614f-4903-b46c-d995324d6517" />


---

## Remove-OffboardUser.ps1

Offboards a user account in three steps: disable the account, strip all group memberships, move to the Disabled OU. The account is not deleted and is retained for 6 months before permanent removal. Prompts for confirmation before making any changes.

**Decision flow:**

1. Check if the account exists in AD, exit if not found
2. Prompt for confirmation before making any changes
3. Disable the account, exit if this step fails to avoid partial offboarding
4. Strip all group memberships, log any individual group removals that fail
5. Move the account to the Disabled OU, flag for manual relocation if this step fails
6. Print a verification summary showing the account is disabled, groups are empty, and the DN points to the Disabled OU

> Note: Domain Users is a primary group and cannot be removed via MemberOf. This is expected behaviour and does not require manual intervention.

<img width="978" height="585" alt="image" src="https://github.com/user-attachments/assets/fa480e54-6fbc-4b9e-af3f-ab6a575141d5" />


---

## Get-ADReports.ps1

An interactive menu-driven tool for AD reporting and account management. Run the script, pick an option, view the results, and optionally export to CSV. All exports are saved to `C:\Scripts\AD\` with the report name and date in the filename.

**Menu options:**

| Option | Name | Description |
|---|---|---|
| 1 | Inactive Users | Users who have not logged in within X days. Offers to disable accounts after results. |
| 2 | Group Members | All members of a specified group. |
| 3 | Locked Accounts | All currently locked out accounts. Offers to unlock directly from results. |
| 4 | Expiring Passwords | Users whose passwords expire within X days. Colour-coded by urgency. |
| 5 | Account Details Lookup | Full summary for a specific user including enabled state, lock status, bad logon count, password expiry, group memberships, and OU. |
| 6 | Recently Created Accounts | Accounts created in the last X days. |
| 7 | Recently Offboarded Accounts | All accounts in the Disabled OU with last modified date for tracking the 6-month retention window. |
| 8 | Export All Reports | Runs reports 1, 3, 4, and 7 silently and exports all results to CSV in one pass. |
| 9 | Exit | Exits the menu. |

**Decision flow:**

1. Display the menu and prompt for a selection
2. Validate the input is a number within the valid range, re-prompt if not
3. Run the selected report and display results
4. If the report supports an action (disable, unlock), prompt before making any changes
5. Offer a CSV export after results are displayed
6. Return to the menu after each report until the user selects Exit

<img width="559" height="357" alt="image" src="https://github.com/user-attachments/assets/7433185d-31cb-4c29-880c-2a8a8c6fd689" />

<img width="939" height="571" alt="image" src="https://github.com/user-attachments/assets/f647e4d1-87ca-40c7-b773-0120b809e202" />

<img width="809" height="531" alt="image" src="https://github.com/user-attachments/assets/2e6dc731-c42d-492b-969e-51979b513ae7" />

<img width="960" height="418" alt="image" src="https://github.com/user-attachments/assets/ab963b7c-cf44-421e-9c08-4cdfc972752d" />




---

## Next Steps

1. Configure Windows Server Backup for AD DS System State
2. Document authoritative vs non-authoritative restore procedure
3. Test restore in lab and produce runbook in markdown
