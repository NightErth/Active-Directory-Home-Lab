# Active Directory Home Lab

A hands-on home lab built to simulate a real-world enterprise Active Directory environment. This project covers the full lifecycle of AD administration — from initial domain setup and user management to Group Policy enforcement and PowerShell automation — demonstrating practical skills in identity management, access control, and Windows Server administration.

---

## Table of Contents

- [Overview](#overview)
- [Technologies Used](#technologies-used)
- [Lab Architecture](#lab-architecture)
- [Step 1 — Active Directory Fundamentals](#step-1--active-directory-fundamentals)
- [Step 2 — Windows Server 2022 Setup](#step-2--windows-server-2022-setup)
- [Step 3 — Installing Windows Server 2022](#step-3--installing-windows-server-2022)
- [Step 4 — Configuring Active Directory Domain Services](#step-4--configuring-active-directory-domain-services)
- [Step 5 — Creating Domain Users](#step-5--creating-domain-users)
- [Step 6 — Joining a Windows 11 Client to the Domain](#step-6--joining-a-windows-11-client-to-the-domain)
- [Step 7 — Organizational Units, Groups & Shared Folders](#step-7--organizational-units-groups--shared-folders)
- [Step 8 — Group Policy Objects (GPOs)](#step-8--group-policy-objects-gpos)
- [Step 9 & 10 — PowerShell Scripting & Automation](#step-9--10--powershell-scripting--automation)
- [Step 11 — Account Lockout Policy & Password Reset](#step-11--account-lockout-policy--password-reset)

---

## Overview

This lab replicates a small enterprise network using VirtualBox, running a Windows Server 2022 Domain Controller and a Windows 11 client machine on an isolated NAT network. The goal is to practice core Active Directory administration tasks in a safe, controlled environment.

---

## Technologies Used

- Windows Server 2022 (Domain Controller)
- Windows 11 (Domain-joined client)
- Oracle VirtualBox
- Active Directory Domain Services (AD DS)
- Active Directory Certificate Services (AD CS)
- Group Policy Management Console (GPMC)
- PowerShell

---

## Lab Architecture

```
VirtualBox NAT Network: ADNetwork
│
├── Domain Controller (Windows Server 2022)
│   ├── AD DS — lab.local
│   ├── AD CS
│   ├── Static IP (DNS points to localhost)
│   └── Shared Folders (SMB via NETLOGON & EngShare)
│
└── Client Machine (Windows 11)
    ├── Domain-joined to lab.local
    └── Static IP (DNS points to DC IP)
```

---

## Step 1 — Active Directory Fundamentals

### What is Active Directory?

Active Directory (AD) is Microsoft's centralized directory service used to manage identities and resources across an organization. It stores and organizes information about network objects — including users, computers, groups, and printers — and controls who can access what resources and under what conditions.

### Why Active Directory?

AD serves as a centralized Identity and Access Management (IAM) solution, providing:

- A single authoritative source for user authentication and authorization
- Granular, role-based access control over users and resources
- A unified point for backup, recovery, and policy enforcement across the organization

### Physical Components of AD

| Component | Description |
|---|---|
| Domain Controller (DC) | A server running AD DS that authenticates users and enforces policy |
| AD DS Data Store | The `NTDS.dit` database file that stores all directory information |
| Global Catalog Server | Holds a partial replica of all objects in the forest for cross-domain queries |
| Read-Only Domain Controller (RODC) | A DC with a read-only copy of the AD database, used in low-trust locations |

### Logical Components of AD

**Domains**
The foundational unit of an AD structure. A domain (e.g., `lab.local`) holds a collection of objects such as users, computers, and groups, and functions as an administrative and security boundary.

**Organizational Units (OUs)**
Container objects within a domain used to logically group and organize other objects (users, computers, groups). OUs are the lowest-level containers to which you can delegate administrative control and apply Group Policy Objects (GPOs).

**Non-Container Objects**
Leaf objects that represent actual resources: users, computers, groups, printers, and GPOs.

**Domain Trees, Domain Trusts & Forests**

| Concept | Description |
|---|---|
| Domain Tree | A hierarchy of domains that share a contiguous namespace (e.g., `lab.local`, `dev.lab.local`) |
| Domain Trust | A relationship between two domains that allows users in one domain to be authenticated by another |
| Forest | The top-level boundary of an AD environment, containing one or more domain trees that share a common schema and global catalog |

---

## Step 2 — Windows Server 2022 Setup

Downloaded the Windows Server 2022 ISO from Microsoft's official Evaluation Center and configured a new virtual machine in VirtualBox with sufficient CPU, RAM, and storage to function as a Domain Controller.

---

## Step 3 — Installing Windows Server 2022

Installed Windows Server 2022 (Desktop Experience) on the VM with standard configuration. Selected the appropriate edition and completed the initial administrator account setup.

---

## Step 4 — Configuring Active Directory Domain Services

### Installing AD DS & Promoting the Server to a Domain Controller

1. Open **Server Manager** → **Manage** → **Add Roles and Features**
2. Select **Role-based installation** → choose the local server
3. Select **Active Directory Domain Services** → proceed through defaults
4. Check **Restart the destination server automatically if required** → **Install**
5. After installation, click **Promote this server to a domain controller**
6. Select **Add a new forest** → set **Root Domain Name** (e.g., `lab.local`)
7. Set the Directory Services Restore Mode (DSRM) password
8. Accept default settings → **Install** (server will restart)

### Installing Active Directory Certificate Services (AD CS)

AD CS enables the domain to issue and manage digital certificates for secure communication, authentication, and encryption within the environment.

1. **Manage** → **Add Roles and Features** → **Role-based installation** → select the domain
2. Select **Active Directory Certificate Services** → proceed with defaults → **Install**
3. After installation, click **Configure AD CS on the destination server** → accept default settings

---

## Step 5 — Creating Domain Users

1. **Tools** → **Active Directory Users and Computers**
2. Expand the domain (`lab.local`) → select the **Users** container
3. Right-click → **New** → **User** → fill in user details → complete the wizard
4. Repeated to create 4–5 additional domain users for use in subsequent steps

---

## Step 6 — Joining a Windows 11 Client to the Domain

### Network Configuration (VirtualBox)

1. **File** → **Tools** → **Network Manager** → **NAT Networks** → **Create**
2. Name the network `ADNetwork`
3. Assign `ADNetwork` to both VMs via **Settings** → **Network** in VirtualBox

### Domain Controller — Static IP Configuration

1. Open **Network and Internet Settings** → **Change adapter settings**
2. Right-click **Ethernet** → **Properties** → **IPv4**
3. Set a static IP address; configure DNS to point to `127.0.0.1` (localhost)

### Windows 11 Client — Static IP & DNS Configuration

1. Set a static IP on the same subnet as the DC
2. Set the **DNS server** to the Domain Controller's static IP address

### Joining the Domain

1. **Settings** → **Accounts** → **Access work or school** → **Connect**
2. Select **Join this device to a local Active Directory domain**
3. Enter the domain name (`lab.local`) → provide Domain Admin credentials
4. Restart the machine → sign in using any previously created domain user account

---

## Step 7 — Organizational Units, Groups & Shared Folders

### OUs vs. Groups

| | Organizational Units (OUs) | Groups |
|---|---|---|
| **Purpose** | Logical structure, management, and administration | Assigning permissions and access to resources |
| **Scope** | Administrative delegation and GPO application | Resource access control (files, printers, etc.) |
| **Contains** | Users, computers, other OUs | Users, computers, other groups |

### Creating OUs

1. **Tools** → **Active Directory Users and Computers**
2. Right-click the domain name → **New** → **Organizational Unit**
3. Created three OUs: `Engineering`, `Management`, `IT`
4. Moved each domain user into their respective OU

### Creating a Security Group & Shared Folder

1. Right-click the `Engineering` OU → **New** → **Group** → name it `EngShare`
2. Open the `EngShare` group → **Members** tab → **Add** relevant users

**Setting up the SMB Share:**

1. **Server Manager** → **File and Storage Services** → **Shares** → **Tasks** → **New Share**
2. Select **SMB Share – Quick**
3. Set the share name to `EngShare` and note the **remote path to share**
4. Under **Permissions** → **Customize permissions** → **Disable inheritance**
5. Add `EngShare` as the principal with appropriate access → **Create**

**Accessing & Mapping the Share (from Client):**

- Paste the remote share path into File Explorer to access the share
- **This PC** → **Map network drive** → enter the remote path → the share is persistently mapped and visible under **This PC**

> **Note:** There are two group types in AD:
> - **Distribution Groups** — used for email distribution lists; cannot be used to assign permissions
> - **Security Groups** — used to control access to resources such as shared folders, printers, and applications

---

## Step 8 — Group Policy Objects (GPOs)

### What are GPOs?

Group Policy Objects (GPOs) are collections of settings that control the working environment of user accounts and computer accounts. GPOs are linked to AD containers (sites, domains, or OUs) and are processed by the client at startup or login, enabling administrators to enforce consistent configurations — such as security settings, software deployment, and desktop customization — across the organization without manual intervention on each machine.

### Lab Task: Enforcing a Department Wallpaper via GPO

**Staging the wallpaper image on the server:**

1. In **Server Manager** → **File and Storage Services** → **Shares** → **NETLOGON**
2. Right-click → **Open Share** → copy the desired wallpaper image into this share
3. Note the full UNC path to the image (e.g., `\\lab.local\NETLOGON\background.jpg`)

**Creating and linking the GPO:**

1. **Tools** → **Group Policy Management**
2. Navigate to **Forest** → **Domains** → `lab.local` → **Engineering** OU
3. Right-click → **Create a GPO and link it here** → name it `EngBackground` → **OK**

**Configuring the policy:**

1. Right-click `EngBackground` → **Edit**
2. Navigate to **User Configuration** → **Policies** → **Administrative Templates** → **Desktop** → **Desktop Wallpaper**
3. Set to **Enabled** → paste the image UNC path into the **Wallpaper Name** field → **OK**

When users in the Engineering OU sign in, the specified wallpaper is automatically applied and enforced by the domain.

---

## Step 9 & 10 — PowerShell Scripting & Automation

### What is PowerShell?

PowerShell is a cross-platform task automation tool consisting of a command-line shell and scripting language built on .NET. In the context of Active Directory, it provides administrators with the ability to perform bulk operations, automate repetitive tasks, and manage AD objects programmatically — capabilities that are impractical or time-consuming through the GUI alone.

### Lab Tasks

- Practiced core PowerShell cmdlets and AD module commands (`Get-ADUser`, `New-ADUser`, `Set-ADUser`, etc.)
- Wrote a PowerShell script to automate the creation of new Active Directory user accounts, including setting attributes, group memberships, and initial passwords — simulating how sysadmins provision users at scale

---

## Step 11 — Account Lockout Policy & Password Reset

### Implementing an Account Lockout Policy via GPO

A lockout policy mitigates brute-force and credential-stuffing attacks by disabling an account after a defined number of failed login attempts.

1. **Tools** → **Group Policy Management** → `lab.local`
2. Right-click the domain → **Create a GPO and Link it here** → name it `AccLockoutPolicy`
3. Right-click `AccLockoutPolicy` → **Edit**
4. Navigate to **Computer Configuration** → **Policies** → **Windows Settings** → **Security Settings** → **Account Policies** → **Account Lockout Policy**
5. Set **Account Lockout Threshold** to `3` (account locks after 3 failed attempts) → **Apply** → **OK**
6. Right-click `AccLockoutPolicy` → **Enforced** to ensure the policy takes precedence

**Validation:** Attempted three incorrect passwords on a domain user account — the account locked as expected.

### Resetting a Locked User Account

**Via GUI (Active Directory Users and Computers):**

1. **Tools** → **Active Directory Users and Computers**
2. Locate the user in their OU or via the search toolbar
3. Right-click the user → **Properties** → **Account** tab → **Advanced options** → uncheck **Password never expires** if applicable → **Apply**
4. Right-click the user → **Reset Password**
5. Enter a temporary default password
6. Check **Unlock the user's account**
7. Check **User must change password at next logon** → **OK**

The user can now authenticate with the temporary credentials and will be prompted to set a new password upon first login.

**Via PowerShell:**

Also practiced automating this workflow using a PowerShell script, leveraging `Unlock-ADAccount` and `Set-ADAccountPassword` cmdlets to reset and unlock accounts programmatically.

---

## Key Takeaways

- Deployed and configured a fully functional AD domain from scratch in a virtualized environment
- Demonstrated centralized identity and access management using OUs, groups, and GPOs
- Practiced real-world sysadmin tasks: user provisioning, shared folder access control, policy enforcement, and account recovery
- Applied PowerShell scripting to automate manual AD administration tasks, reflecting industry-standard practices
