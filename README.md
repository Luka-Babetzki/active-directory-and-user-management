# Active Directory and User Management

## Overview
Built a Windows Server 2022 domain environment to practise common Help Desk support tasks including user account management, organisational unit structure, shared folder permissions, and Group Policy configuration in a virtualised lab.

## Objectives
- Create a domain and add users to it
- Build out organisational groups and applying RBAC for security on a department basis 
- Execute remote server-side commands such as password resets, etc

## Technologies Used
- Active Directory Domain Services (AD DS)
- Group Policy Object (GPO)
- File and Storage Services (for Network Shares)
- PowerShell

>**Lab Environment:** See my <! = href:"">IT homelab setup documentation</a> for virtualisation platform and base configuration details.

## Architecture
...

## Implementation Steps

### 1. Install AD DS on the Windows Server 2022 (Domain Controller)

**Installation Process:**
1. Opened Server Manager → Manage → Add Roles and Features
2. Proceeded to Server Roles and selected Active Directory Domain Services
3. Completed installation and clicked "Promote this server to a domain controller"
4. Selected "Add a new forest" and set Root domain name as LAB.local
5. Verified NetBIOS domain name was set to LAB
6. Completed configuration wizard and restarted server

**Verifiy:** After restart, login screen displayed LAB\Administrator confirming successful domain controller promotion

### 2. Create Domain Users

**User Creation Process:**
1. Opened Tools → Active Directory Users and Computers (ADUC)
2. Created new OU called "Groups" to organise default security groups
3. Moved existing security groups from Users container into Groups OU for better organisation
4. Created 5 test users in the Users container following naming convention: first initial + surname (e.g., jsmith for John Smith)

**Password Policy Applied:**
- Set temporary initial passwords for all accounts
- Enabled "User must change password at next logon" to enforce secure password practices

User Accounts Created:

|    Account Name    |            Name             |
|--------------------|-----------------------------|
|  `administrator`   |    default domain admin     |
|  `jsmith`          |    John Smith               |
|  `abloggs`         |    Alice Bloggs             |
|  `bwilliams`       |    Bob Williams             |
|  `cjones`          |    Charlie Jones            |
|  `dtaylor`         |    David Taylor             |

### 3. Attach a Windows 11 Client to the Domain

**Domain Join Process:**
1. On Windows 11 client, navigated to Settings → Accounts → Access work or school
2. Clicked "Connect" → "Join this device to a local Active Directory domain"
3. Entered domain name: LAB.local
4. Authenticated using domain administrator credentials
5. Restarted workstation to apply domain membership

**Post-Join Verification:**
1. Logged into Windows 11 client as domain user jsmith
2. User prompted to reset password on first login (as configured)
3. Verified on DC01 under Computers container that the Windows 11 client appeared

### 4. Using OUs for Organisational Hierarchy and Creating Network Shares

**OU Structure for Departmental Organisation:**
- Created three OUs to simulate a small startup's structure:

|      OU        |      Account Name          |
|----------------|----------------------------|
| IT             | `administrator` + `jsmith` |
| Management     | `abloggs` + `bwilliams`    |
| Engineering    | `cjones` + `dtaylor`       |

- Moved users from the default Users container into their respective departmental OUs.

**Implementation & Configuration Steps for Shared Folder:**

1. Created Security Group:
- In Engineering OU, created security group: EngineeringShare
- Added members: cjones, dtaylor (Engineering), abloggs (Management)

2. Created SMB Share:
- Opened Server Manager → File and Storage Services → Shares
- Selected Tasks → New Share → SMB Share - Quick
- Named share: EngineeringShare
- Share path: C:\Shares\EngineeringShare

3. Configured Permissions:
- Disabled inheritance to prevent default "Everyone" permissions
- Clicked Add → Select a principal → added EngineeringShare security group
- Granted Read/Write/Execute permissions to the group
- Removed all other users/groups

4. Testing Access:
- Logged into Windows 11 client as cjones (member of EngineeringShare group)
- Accessed share via File Explorer: \\DC01\EngineeringShare
- Successfully created test file in shared folder
- Mapped network drive (right-click This PC → Map Network Drive) for persistent access


5. Negative Testing:
- Logged in as jsmith (IT user, not in EngineeringShare group)
- Attempted to access \\DC01\EngineeringShare → Access Denied (expected behaviour)


### 5. Applying Group Policy Objects (GPOs) to Engineering Department

**GPO Configuration for Engineering OU Implementation Steps:**
1. Opened Group Policy Management Console (GPMC)
2. Right-clicked Engineering OU → "Create a GPO in this domain, and Link it here..."
3. Named GPO: Engineering-SecurityPolicy
4. Edited GPO using Group Policy Management Editor

**Password Policy (Computer Configuration):**
- Minimum password length: 10 characters
- Password must meet complexity requirements: Enabled
- Maximum password age: 60 days

**USB Storage Restriction (Computer Configuration):**
- Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access
- Set "Removable Disks: Deny write access" to Enabled

**Testing GPO Application:**
1. Logged into Windows 11 client as Engineering user
2. Ran gpupdate /force to apply policies immediately
3. Ran gpresult /r to verify Engineering-SecurityPolicy was applied
4. Tested USB restriction by attempting to write to USB drive → Access Denied (success)
5. Attempted password change with 8 characters → Rejected due to 10-character minimum (success)


### 6. Additional (Onboarding Script)
...

## Key Learnings
- How to create a domain and attach client machines to it
- How to add and configure users using organisational units (OUs) and group policy objects (GPO)
- How to implement network shares that cross shares resources over OUs

## Future Enhancements
- [ ] Add network architecture diagram showing VM topology
- [ ] Add screenshots illustrating VM configuration steps
- [ ] Write the Onboarding Script

## Resources
- [Microsoft AD DS Documentation](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services)
- [Group Policy Management Guide](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview)
- [PowerShell Active Directory Module](https://learn.microsoft.com/en-us/powershell/module/activedirectory/)
- [Windows Server 2022 ISO](https://www.microsoft.com/en-gb/evalcenter/evaluate-windows-server-2022)
