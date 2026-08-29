# Active Directory: Network File Shares and Permissions

## Project Overview

In this lab, I created and secured network file shares in an Active Directory environment hosted in Microsoft Azure.

I configured different access levels for domain users and administrators, tested the permissions from a Windows client, created an Active Directory security group, and used group membership to grant access to a restricted accounting share.

This lab demonstrated how organizations use shared folders and security groups to provide users with access based on their job responsibilities.

## Environments and Technologies Used

- Microsoft Azure
- Windows Server 2022
- Windows 10
- Active Directory Domain Services
- Active Directory Users and Computers
- Server Message Block file sharing
- Windows share permissions
- NTFS permissions
- Remote Desktop Protocol

## Prerequisites

The following Active Directory environment was already available:

- **DC-1:** Windows Server 2022 domain controller and file server
- **Client-1:** Windows 10 computer joined to the domain
- Domain administrator account available
- Standard domain user account available
- Successful network communication between Client-1 and DC-1

Both Azure virtual machines were started before beginning the lab.

## Part 1: Sign In to the Virtual Machines

I connected to DC-1 using the domain administrator account and connected to Client-1 using a standard domain user account.

This allowed me to configure the shares as an administrator and then test the permissions from the perspective of a regular employee.

## Part 2: Create the Shared Folders

On the `C:\` drive of DC-1, I created the following folders:

```text
C:\read-access
C:\write-access
C:\no-access
C:\accounting
```

Each folder was used to demonstrate a different access-control scenario.

![Folders Created on DC-1] <img width="2200" height="1429" alt="6E8CE3AE-C1E3-4A18-9082-926CE50D2875_1_102_o" src="https://github.com/user-attachments/assets/b259939a-bfdd-4208-a8be-18a1209c4f65" />


## Part 3: Configure the Read-Access Share

I shared the `read-access` folder with the following permissions:

| Setting | Configuration |
|---|---|
| Folder | `read-access` |
| Security principal | Domain Users |
| Permission | Read |

This configuration allowed standard domain users to open and view files but prevented them from creating, modifying, or deleting content.

![Read-Access Permissions] <img width="2200" height="1429" alt="353E9B33-64C7-43CC-94AF-8A21F75EC9AB_1_102_o" src="https://github.com/user-attachments/assets/2bb37468-cd4c-4efc-bcef-5a35f20f2e24" />


## Part 4: Configure the Write-Access Share

I shared the `write-access` folder with the following permissions:

| Setting | Configuration |
|---|---|
| Folder | `write-access` |
| Security principal | Domain Users |
| Permission | Read/Write |

This configuration allowed standard domain users to view existing content and create or modify files inside the share.



## Part 5: Configure the No-Access Share

I shared the `no-access` folder with the following permissions:

| Setting | Configuration |
|---|---|
| Folder | `no-access` |
| Security principal | Domain Admins |
| Permission | Read/Write |

Because the standard test user was not a member of Domain Admins, the user was unable to access this share.


## Part 6: Access the Shares from Client-1

While signed in to Client-1 as a standard domain user, I opened the Run dialog and entered the Universal Naming Convention path to DC-1:

```text
\\DC-1
```

This displayed the shared folders published by DC-1.


## Part 7: Test the Share Permissions

I tested each shared folder from Client-1 and observed the following behavior:

| Share | Access Result | File Creation Result |
|---|---|---|
| `read-access` | User could open the folder | User could not create or modify files |
| `write-access` | User could open the folder | User could create and modify files |
| `no-access` | Access was denied | User could not create or modify files |

These results confirmed that the configured permissions were working correctly.

![Share Permission Testing] <img width="2200" height="1429" alt="BB1ED792-E572-4222-A859-CD17EFD86453_1_102_o" src="https://github.com/user-attachments/assets/48b27c42-8ca6-43d0-a6c2-e284c4e1ce94" />

## Part 8: Create the Accountants Security Group

On DC-1, I opened **Active Directory Users and Computers** and created a new security group:

| Setting | Configuration |
|---|---|
| Group name | ACCOUNTANTS |
| Group type | Security |
| Group scope | Global |

Security groups allow administrators to assign permissions to a group instead of configuring each user individually.

![Accountants Security Group] <img width="2200" height="1429" alt="2197B266-3F7E-467B-BE41-C7EFCF0E1C5C_1_102_o" src="https://github.com/user-attachments/assets/99a21e93-31e4-4d24-921a-274b2c164d6b" />

## Part 9: Configure the Accounting Share

I shared the `accounting` folder and assigned the following permissions:

| Setting | Configuration |
|---|---|
| Folder | `accounting` |
| Security principal | ACCOUNTANTS |
| Permission | Read/Write |

Only users who were members of the ACCOUNTANTS security group were intended to access and modify content inside this share.



## Part 10: Test Access Before Group Membership

I returned to Client-1 and attempted to access the accounting share while signed in as the standard domain user:

```text
\\DC-1\accounting
```

The attempt failed because the user was not a member of the ACCOUNTANTS security group.

![Accounting Access Denied] <img width="2200" height="1429" alt="38D9184B-6121-4170-8BB3-9A01A71A5C4F_1_102_o" src="https://github.com/user-attachments/assets/51455b2e-40c8-4faa-bced-d8a1196e32f4" />

## Part 11: Add the User to the Accountants Group

On DC-1, I returned to Active Directory Users and Computers and added the standard test user to the ACCOUNTANTS security group.

The user then signed out of Client-1 and signed back in. Signing in again created a new access token containing the user’s updated security-group membership.

![User Added to Accountants] <img width="2200" height="1429" alt="3621E3DA-13D2-4A1D-AD45-4E39EC4018FC_1_102_o" src="https://github.com/user-attachments/assets/eed5ec98-f60e-432e-b629-d41d6e2fb07d" />


## Part 12: Verify Access After Group Membership

After signing back in, the user accessed:

```text
\\DC-1\accounting
```

The user was now able to open the accounting share and create files because the account inherited the Read/Write permissions assigned to the ACCOUNTANTS group.

This confirmed that access could be managed effectively through Active Directory security-group membership.



## Permission Summary

| Shared Folder | Authorized Group | Permission | Standard User Result |
|---|---|---|---|
| `read-access` | Domain Users | Read | Can view but cannot modify |
| `write-access` | Domain Users | Read/Write | Can view and modify |
| `no-access` | Domain Admins | Read/Write | Access denied |
| `accounting` | ACCOUNTANTS | Read/Write | Access granted after group membership |

## Share and NTFS Permissions

Windows file access can be controlled by two permission layers:

- **Share permissions:** Apply when a folder is accessed over the network.
- **NTFS permissions:** Apply to files and folders stored on the drive, whether accessed locally or over the network.

When both permission types are used, a user’s effective network access is determined by the most restrictive combination of the applicable share and NTFS permissions.

## Security Principle Demonstrated

This lab demonstrated the principle of least privilege. Users were given only the access required for their responsibilities:

- Domain users received read-only access where modification was unnecessary.
- Domain users received read/write access where collaboration was permitted.
- Administrative content was restricted to Domain Admins.
- Accounting content was restricted to members of the ACCOUNTANTS security group.

Using security groups instead of assigning permissions directly to individual accounts simplifies administration and makes access easier to review.

## Troubleshooting Access Problems

When troubleshooting access to a network share, I would verify:

1. The client can communicate with the file server.
2. The shared folder exists and is published.
3. The user is signed in with the expected domain account.
4. The correct security group has permission to access the share.
5. The user belongs to the required security group.
6. The NTFS permissions allow the requested action.
7. The user has signed out and back in after a group-membership change.

## Skills Demonstrated

- Creating network file shares
- Configuring read and read/write permissions
- Restricting access to administrative groups
- Accessing shares with UNC paths
- Testing effective permissions from a client computer
- Creating Active Directory security groups
- Managing group membership
- Applying role-based access to shared resources
- Understanding share and NTFS permissions
- Troubleshooting network-share access
- Applying the principle of least privilege



## Conclusion

In this lab, I created multiple network file shares and configured different access levels for domain users, domain administrators, and a department-specific security group.

I verified read-only, read/write, and denied-access behavior from Client-1. I also demonstrated how adding a user to the ACCOUNTANTS security group granted access to a restricted departmental share.

This lab provided practical experience with Windows file sharing, Active Directory security groups, access control, permission testing, and least-privilege administration.
