# Lab 1 — Active Directory
### Windows Server 2025 · Azure Free Account · Identity & Access Management
Watch me do this lab !!
[
https://www.loom.com/share/2b2b9cb05cbc412b90a5dd8a91e6dd07)
![Active Directory](https://img.shields.io/badge/Windows_Server-2025-0078D4?style=flat&logo=windows&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-Free_Tier-0089D6?style=flat&logo=microsoft-azure&logoColor=white)
![Cost](https://img.shields.io/badge/Cost-$0-brightgreen?style=flat)
![Duration](https://img.shields.io/badge/Duration-3--5_hours-yellow?style=flat)

---

## Overview

| Field | Value |
|-------|-------|
| **Certification Alignment** | CompTIA Network+ · Security+ · Azure Administrator |
| **Free Tools** | Windows Server 2025 Evaluation (180 days) · Azure Free Account |
| **Time to Complete** | 3–5 hours across multiple sessions |
| **Estimated Cost** | $0 — fully covered by free tiers and evaluation licences |
| **Career Relevance** | IT Support · Sysadmin · Cloud Engineer · Security Analyst |

Active Directory is the identity backbone of virtually every enterprise Windows environment. It controls which users can log into which computers, which groups can access which file shares, and which policies apply to which parts of the organisation. This lab builds a working AD environment from scratch — promoting a domain controller, creating an OU structure, managing users and groups, and enforcing policy via GPOs.

---

## Architecture

```
┌─────────────────────────── Forest: lab.local ────────────────────────────────┐
│                                                                               │
│              ┌───────────────────────────────────┐                           │
│              │         Domain Controller          │                           │
│              │   Windows Server 2025 · lab.local  │                           │
│              └────────────────┬──────────────────┘                           │
│                               │                                               │
│   ┌─────────────────────── Domain: lab.local ──────────────────────────┐     │
│   │                           │                          ┌───────────┐ │     │
│   │          ┌────────────────┼────────────────┐         │    GPO    │ │     │
│   │          │                │                │         │IT Security│ │     │
│   │    ┌─────▼──┐  ┌──────────▼─┐  ┌────────▼─┐         │  Policy   │ │     │
│   │    │OU: IT  │  │OU: Finance │  │  OU: HR  │ ┌──────┐ └─────┬────┘ │     │
│   │    │alice   │  │ bob.patel  │  │  carol   │ │Sales │       │      │     │
│   │    └───┬────┘  └─────┬──────┘  └────┬─────┘ └──┬───┘  linked to  │     │
│   │        │             │              │           │    IT OU ←──────┘│     │
│   │   ┌────▼────┐  ┌─────▼──────┐ ┌────▼───┐ ┌────▼──────┐           │     │
│   │   │IT_Admins│  │Finance_    │ │HR_Users│ │Sales_Users│           │     │
│   │   │ (group) │  │Users(group)│ │(group) │ │  (group)  │           │     │
│   │   └─────────┘  └────────────┘ └────────┘ └───────────┘           │     │
│   │                                                                    │     │
│   │              ┌──────────────────────────────┐                     │     │
│   │              │       OU: Computers           │                     │     │
│   │              │   Domain-joined workstations  │                     │     │
│   │              └──────────────────────────────┘                     │     │
│   └────────────────────────────────────────────────────────────────────┘     │
│                                                                               │
│        ┌─────────────────────┐        ┌─────────────────────┐                │
│        │  Azure (Option A)   │──RDP──▶│  VirtualBox (B)     │                │
│        │  Standard_B2s       │        │  Local · 4GB RAM    │                │
│        └─────────────────────┘        └─────────────────────┘                │
└───────────────────────────────────────────────────────────────────────────────┘
```

> **Flow**: The Domain Controller is the identity authority for `lab.local`. Users live in department OUs and inherit permissions through Security Group membership (RBAC). The IT Security Policy GPO is linked to the IT OU, enforcing password and lockout rules on every machine and user in that container. Workstations join the domain and have their computer accounts placed in the Computers OU.

---

## Career Relevance

| Role | Application |
|------|-------------|
| **IT Support / Help Desk** | Password resets, account unlocks, group membership changes — the top three ticket types in any enterprise |
| **Sysadmin** | Designing OU structure, deploying GPOs, managing domain-joined machines at scale |
| **Cloud Engineer** | Entra ID (cloud AD) uses the same concepts: users, groups, roles, conditional access. On-prem knowledge transfers directly |
| **Security Analyst** | AD is the most targeted system in ransomware attacks. Understanding how it works is the foundation of defending it |

---

## Skills Covered

| Skill | Real-World Application |
|-------|----------------------|
| Promote a Windows Server to Domain Controller | The first step in every enterprise Windows environment |
| Create Organisational Units (OUs) | Apply different policies to different departments centrally |
| Create users, groups, and group memberships | Every access decision in an enterprise is group-based |
| Configure Group Policy Objects (GPOs) | Enforce settings across every machine in the domain |
| Join a machine to the domain | Connecting a workstation as a managed, policy-enforced resource |
| Configure role-based access with security groups | Principle of least privilege applied practically |
| Reset passwords and manage account lifecycle | The most frequent real-world task for IT support |

---

## Lab Steps

### Step 1 — Get the Free Resources

**Option A — Azure (Recommended)**

1. Create a free account at [azure.microsoft.com/free](https://azure.microsoft.com/free)
2. Sign in to [portal.azure.com](https://portal.azure.com) and create a Virtual Machine with the settings below

| Setting | Value |
|---------|-------|
| Region | East US |
| Image | Windows Server 2025 Datacenter — Gen2 |
| Size | Standard_B2s (2 vCPU, 4GB RAM) |
| Authentication | Password |
| Public inbound ports | Allow RDP (3389) |
| OS disk | Standard SSD |

> **Cost tip:** Stop (don't delete) the VM after each session. A B2s runs ~$0.05/hour; stopping pauses compute billing and preserves your $200 free credit.

> **Clipboard fix:** Download the RDP file from the Azure portal (Connect → Download RDP File) and open it with the native Remote Desktop app. In the RDP client, go to Show Options → Local Resources → check **Clipboard**. This enables copy/paste between your local machine and the VM.

**Option B — VirtualBox (Local)**

- Download [VirtualBox](https://www.virtualbox.org) and the [Windows Server 2025 Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/)
- Create a VM: 4GB RAM, 60GB disk, Windows Server 2019/2022 type
- Minimum host requirements: 8GB RAM, quad-core CPU with virtualisation enabled in BIOS

---

### Step 2 — Install Active Directory Domain Services

In Server Manager: **Manage → Add Roles and Features → Server Roles → Active Directory Domain Services**

```powershell
# Install AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Install Group Policy Management Console (needed for Step 5)
Install-WindowsFeature -Name GPMC
```

---

### Step 3 — Promote the Server to a Domain Controller

In Server Manager, click the **yellow warning flag → Promote this server to a domain controller → Add a new forest → Root domain name: `lab.local`**

```powershell
Import-Module ADDSDeployment
Install-ADDSForest `
  -DomainName 'lab.local' `
  -DomainNetBiosName 'LAB' `
  -InstallDns:$true `
  -SafeModeAdministratorPassword (ConvertTo-SecureString 'YourDSRMPassword!' -AsPlainText -Force) `
  -Force:$true
```

The server restarts automatically. You have now created a new Active Directory forest (`lab.local`) — this server is the root domain controller and authoritative DNS source for all identity decisions.

---

### Step 4 — Build the Organisational Structure

**Create Organisational Units**

```powershell
New-ADOrganizationalUnit -Name "IT"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Finance"   -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "HR"        -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Sales"     -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Computers" -Path "DC=lab,DC=local"
```

**Create Security Groups**

```powershell
New-ADGroup -Name "IT_Admins"     -GroupScope Global -GroupCategory Security -Path "OU=IT,DC=lab,DC=local"
New-ADGroup -Name "Finance_Users" -GroupScope Global -GroupCategory Security -Path "OU=Finance,DC=lab,DC=local"
New-ADGroup -Name "HR_Users"      -GroupScope Global -GroupCategory Security -Path "OU=HR,DC=lab,DC=local"
New-ADGroup -Name "Sales_Users"   -GroupScope Global -GroupCategory Security -Path "OU=Sales,DC=lab,DC=local"
```

**Create User Accounts and Assign Group Memberships**

> **Important:** Run the entire block below at once — the `$password` variable must be defined before the `New-ADUser` commands. Select all, then paste into PowerShell and press Enter.

```powershell
# Step 1 — define password variable first
$password = ConvertTo-SecureString "Welcome@2026!" -AsPlainText -Force

# Step 2 — create users
New-ADUser -Name "alice.chen" -GivenName "Alice" -Surname "Chen" `
  -SamAccountName "alice.chen" -UserPrincipalName "alice.chen@lab.local" `
  -Path "OU=IT,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "bob.patel" -GivenName "Bob" -Surname "Patel" `
  -SamAccountName "bob.patel" -UserPrincipalName "bob.patel@lab.local" `
  -Path "OU=Finance,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "carol.jones" -GivenName "Carol" -Surname "Jones" `
  -SamAccountName "carol.jones" -UserPrincipalName "carol.jones@lab.local" `
  -Path "OU=HR,DC=lab,DC=local" -AccountPassword $password -Enabled $true

New-ADUser -Name "david.smith" -GivenName "David" -Surname "Smith" `
  -SamAccountName "david.smith" -UserPrincipalName "david.smith@lab.local" `
  -Path "OU=Sales,DC=lab,DC=local" -AccountPassword $password -Enabled $true

# Step 3 — assign group memberships
Add-ADGroupMember -Identity "IT_Admins"     -Members "alice.chen"
Add-ADGroupMember -Identity "Finance_Users" -Members "bob.patel"
Add-ADGroupMember -Identity "HR_Users"      -Members "carol.jones"
Add-ADGroupMember -Identity "Sales_Users"   -Members "david.smith"
```

---

### Step 5 — Configure Group Policy

Open **Group Policy Management** from the Tools menu. Right-click the IT OU → **Create a GPO in this domain and link it here** → Name: `IT Security Policy` → Right-click → **Edit**.

| Policy Path | Setting | Value |
|-------------|---------|-------|
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Minimum password length | 12 |
| Computer Config → Windows Settings → Security → Account Policies → Password Policy | Password must meet complexity requirements | Enabled |
| Computer Config → Windows Settings → Security → Local Policies → Security Options | Interactive logon: Machine inactivity limit | 900 seconds |
| Computer Config → Administrative Templates → System → Removable Storage Access | All removable storage classes: Deny all access | Enabled |

> **Test:** Join a second VM to `lab.local`, move its computer account into the IT OU, run `gpupdate /force`, log in as `alice.chen`, and verify the screen lock policy applies.

---

### Step 6 — Common Help Desk Tasks

**Reset a password**
```powershell
Set-ADAccountPassword -Identity "bob.patel" -Reset `
  -NewPassword (ConvertTo-SecureString "NewPass@2026!" -AsPlainText -Force)
Set-ADUser -Identity "bob.patel" -ChangePasswordAtLogon $true
```

**Unlock a locked account**
```powershell
Unlock-ADAccount -Identity "carol.jones"
```

**Disable an account (offboarding)**
```powershell
Disable-ADAccount -Identity "david.smith"

# Find all disabled accounts
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

**Audit — find inactive accounts**
```powershell
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} `
  -Properties LastLogonDate | Select-Object Name, LastLogonDate

# Check group membership for a user
Get-ADPrincipalGroupMembership -Identity "alice.chen" | Select-Object Name
```

---

## Verification

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Domain controller is running | `Get-ADDomainController` | Returns DC info including forest `lab.local` |
| OUs exist | `Get-ADOrganizationalUnit -Filter *` | Lists all 5 OUs |
| Users exist and are enabled | `Get-ADUser -Filter {Enabled -eq $true}` | Lists your 4 test accounts |
| Group memberships correct | `Get-ADGroupMember -Identity IT_Admins` | Returns `alice.chen` |
| GPO is linked | `Get-GPInheritance -Target 'OU=IT,DC=lab,DC=local'` | Shows `IT Security Policy` as linked |

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| PowerShell prompts for `Name:` when creating users | Run the **entire script block at once** — the `$password` line must come first |
| Cannot copy/paste into the VM | Open RDP client → Show Options → Local Resources → check **Clipboard**. Reconnect. |
| Promotion fails: DNS conflict | Set the NIC's preferred DNS to `127.0.0.1` before promoting |
| Cannot RDP after domain join | Log in as `LAB\Administrator`, not just `Administrator` |
| GPO not applying | Run `gpupdate /force`, then `gpresult /r` to see applied policies |
| User cannot log in after creation | Confirm the account is **Enabled** and `ChangePasswordAtLogon` is set correctly |
| AD Users and Computers not showing | Run `dsa.msc` from the Run dialog |

---

## Key Concepts

**Domain Controller** — The server that runs Active Directory. All authentication decisions on the domain flow through it. Credentials are checked here every time a user logs in anywhere on the domain.

**Forest & Domain** — A Forest is the top-level container for your entire AD structure. A Domain (`lab.local`) is the boundary inside the forest — everything inside it is managed together.

**Organisational Unit (OU)** — A folder inside Active Directory. OUs organise users and computers by department or function, and are the attachment point for Group Policies.

**Security Group** — A container for user accounts. Access to resources is granted to groups, not individuals — this is role-based access control (RBAC) in practice.

**Group Policy Object (GPO)** — A rulebook Windows enforces automatically. Link one GPO to an OU and every machine/user inside it gets those settings on the next login or `gpupdate`.

---

*Lab design aligned to CompTIA Network+, Security+, and Microsoft Azure Administrator (AZ-104) objectives.*
