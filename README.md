# AD-002 — Active Directory User Lifecycle Management

## Onboarding & Offboarding

## Project Overview

This project demonstrates a complete Active Directory user lifecycle workflow in a Windows domain environment.

The lab covers:

- Creating a new domain user
- Placing the user in the correct Organizational Unit (OU)
- Assigning group-based access
- Validating domain authentication
- Verifying effective group membership
- Disabling a departing employee account
- Removing access
- Moving the account into an offboarding OU
- Validating that authentication and access have been revoked
- Troubleshooting Active Directory connectivity during validation

---

## Environment

### Domain

`LAB.local`

### Domain Controller

`DC01`

### Domain-Joined Workstation

`WS01`

### Technologies Used

- Windows Server
- Active Directory Domain Services (AD DS)
- Active Directory Users and Computers (ADUC)
- DNS
- Windows domain authentication
- Security groups
- Organizational Units
- Command Prompt
- VirtualBox

---

## Scenario

Human Resources submitted a request to onboard a new employee into the Engineering department.

### New Employee

- **Name:** Jordan Miles
- **Username:** `jmiles`
- **Department:** Engineering
- **Domain:** `LAB.local`

The account needed to be created, placed in the appropriate Organizational Unit, assigned Engineering access, and validated from a domain-joined workstation.

Later, HR submitted an employee termination request.

The account needed to be disabled, access removed, moved into the organization's offboarding structure, and verified to ensure that the former employee could no longer authenticate.

---

# Part 1 — New Hire Onboarding

## Step 1 — Create Engineering Security Group

A dedicated Active Directory security group was created for Engineering employees.

`Engineering_SG`

Configuration:

- **Group scope:** Global
- **Group type:** Security

Using group-based access instead of assigning permissions directly to individual users allows access to be managed centrally.

The access model used was:

User  
↓  
Security Group  
↓  
Department Resources

This approach makes onboarding, role changes, and offboarding easier to manage.

### Screenshot

![Engineering Security Group](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/01-engineering-security-group.png)

---

## Step 2 — Create New Employee Account

A new domain account was created for:

**Jordan Miles**

Logon name:

`jmiles`

The account was created inside the:

`Engineering`

Organizational Unit.

The user was configured to change the temporary password during the initial sign-in.

### Screenshot

![Jordan Miles in Engineering OU](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/02-jordan-engineering-ou.png)

---

## Step 3 — Assign Engineering Access

Jordan Miles was added to:

`Engineering_SG`

This granted the user Engineering-related access through group membership rather than direct user permissions.

### Screenshot

![Jordan Miles Engineering Group Membership](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/03-engineering-group-membership.png)


---

## Step 4 — Validate Domain Authentication

The employee logged into the domain-joined workstation using:

`LAB\jmiles`

Authentication was validated with:

`whoami`

Expected result:

`lab\jmiles`

### Screenshot

---

## Step 5 — Validate Domain Account and Group Membership

The following command was used:

`net user jmiles /domain`

This verified that the account existed in the domain and returned the user's Active Directory information.

Effective group membership was also verified using:

`whoami /groups`

A filtered query was also used:

`whoami /groups | findstr /i engineering`

The output confirmed membership in:

`LAB\Engineering_SG`

### Screenshot

![Domain Account and Engineering Group Validation](images/AD-002/05-domain-group-validation.png)

---

# Troubleshooting During Onboarding

During validation, the workstation returned:

`System error 1355`

`The specified domain either does not exist or could not be contacted.`

The workstation was still able to use cached domain credentials, but Active Directory queries could not reach the domain controller.

Network configuration was reviewed using:

`ipconfig /all`

The workstation was configured with:

- **IP Address:** 
- **DNS Server:** 

Additional testing included:

`echo %logonserver%`

and:

`ping 10.0.2.4`

The issue was traced to the Domain Controller VM being powered off.

After starting `DC01`, connectivity was restored and Active Directory queries completed successfully.

This demonstrated an important Active Directory troubleshooting concept:

> Cached domain credentials may allow a user to sign in even when the domain controller is temporarily unavailable.

### Screenshot



---

# Part 2 — Employee Offboarding

HR later submitted a request indicating that Jordan Miles was leaving the organization.

The objective was to revoke access while preserving the Active Directory object for administrative and audit purposes.

---

## Step 1 — Create Offboarded Users OU

A dedicated Organizational Unit was created:

`Offboarded Users`

The OU was configured with protection against accidental deletion.

Separating disabled accounts from active employees makes account management and auditing easier.

---

## Step 2 — Disable User Account

The account:

`jmiles`

was disabled in Active Directory.

Disabling the account prevents normal domain authentication while preserving the user object.

### Screenshot

![Jordan Miles Account Disabled](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/07-account-disabled.png)

---

## Step 3 — Move User to Offboarded Users OU

Jordan Miles was moved from:

`Engineering`

to:

`Offboarded Users`

This separated the former employee from active Engineering personnel.

### Screenshot

![Jordan Miles Moved to Offboarded Users OU](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/08-offboarded-users-ou.png)


---

## Step 4 — Remove Engineering Access

Jordan Miles was removed from:

`Engineering_SG`

This removed Engineering-specific group access.

The resulting membership was limited to default domain membership such as:

`Domain Users`

---

## Step 5 — Validate Account Lockout

A login attempt was performed from the domain workstation using:

`LAB\jmiles`

The authentication attempt failed because the Active Directory account was disabled.

This confirmed that the employee could no longer sign into the domain.

### Screenshot

![Failed Login After Account Disablement](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/09-disabled-login-test.png)


---

## Step 6 — Validate Group Removal

The following command was run:

`net user jmiles /domain`

The output was reviewed to verify that:

`Engineering_SG`

was no longer listed under the user's global group memberships.

### Screenshot

![Engineering Group Removed From Jordan Miles](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/10-group-removal-validation.png)


---

## Step 7 — Final Active Directory Validation

The user object was verified inside:

`Offboarded Users`

The account remained disabled and no longer contained Engineering-specific group membership.

Final state:

- **User:** Jordan Miles
- **Account:** Disabled
- **OU:** Offboarded Users
- **Engineering_SG:** Removed
- **Domain authentication:** Blocked

### Screenshot

![Final Offboarding State](AD-002-Active-Directory-User-Lifecycle-Management%20IMAGES/11-final-offboarding-state.png)

---

# User Lifecycle Workflow

HR New Hire Request  
↓  
Create Domain Account  
↓  
Assign Organizational Unit  
↓  
Assign Security Group  
↓  
Validate Domain Login  
↓  
Validate Group Access  
↓  
Employee Departure  
↓  
Disable Account  
↓  
Remove Security Group Access  
↓  
Move to Offboarded Users OU  
↓  
Validate Authentication Failure  
↓  
Validate Access Removal

---

# Commands Used

`whoami`

`whoami /groups`

`whoami /groups | findstr /i engineering`

`net user jmiles /domain`

`ipconfig /all`

`echo %logonserver%`

`ping 

---

# Security Concepts Demonstrated

## Group-Based Access Control

Access was assigned through a security group rather than directly to the user.

## Least Privilege

The employee received only the access required for the Engineering role.

## Identity Lifecycle Management

The project covered both account provisioning and account deprovisioning.

## Access Revocation

Offboarding included both account disablement and removal of assigned group access.

## Organizational Unit Management

Users were separated based on their lifecycle state.

## Authentication Validation

Domain authentication was tested from a domain-joined endpoint.

## Active Directory Troubleshooting

Domain connectivity, DNS configuration, domain controller availability, and cached credentials were investigated during the lab.

---

# Skills Demonstrated

- Active Directory administration
- User provisioning
- User deprovisioning
- Security group administration
- Organizational Unit management
- Windows domain authentication
- DNS troubleshooting
- Active Directory troubleshooting
- Identity lifecycle management
- Access control
- Least privilege
- Technical documentation

---

# Outcome

The employee lifecycle was successfully managed from initial account provisioning through final access revocation.

The lab demonstrated how Active Directory can be used to manage users according to their employment lifecycle while maintaining group-based access control, centralized administration, and clear validation of both provisioning and deprovisioning actions.

---


