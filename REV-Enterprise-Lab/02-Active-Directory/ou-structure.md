# Organizational Unit Structure

## 📋 Description

This document outlines the comprehensive Organizational Unit (OU) structure for the REV-Enterprise-Lab Active Directory environment, designed to provide logical organization, simplified administration, and effective Group Policy application.

## 🎯 Objectives

- Create a hierarchical OU structure for logical organization
- Enable efficient Group Policy inheritance and application
- Support delegation of administrative tasks
- Facilitate departmental resource management
- Ensure scalability for future growth

## 🏗️ OU Design Principles

### Design Strategy
- **Departmental Organization**: Group resources by business function
- **Geographic Separation**: Separate resources by location (future expansion)
- **Administrative Delegation**: Enable departmental self-service
- **Policy Application**: Optimize Group Policy inheritance
- **Security Boundaries**: Implement least privilege access

### Naming Conventions
- **OUs**: Use descriptive names (e.g., "Human Resources", "IT Department")
- **Sub-OUs**: Use functional names (e.g., "Users", "Computers", "Groups")
- **Special OUs**: Use prefix for special purposes (e.g., "Admin-", "Temp-")

## 📊 OU Structure Diagram

```
rev.local
├── 🏢 Administrative
│   ├── Admin-Accounts
│   ├── Service-Accounts
│   └── Admin-Workstations
├── 🏢 Departments
│   ├── Human Resources
│   │   ├── HR-Users
│   │   ├── HR-Computers
│   │   └── HR-Groups
│   ├── Housekeeping
│   │   ├── HK-Users
│   │   ├── HK-Computers
│   │   └── HK-Groups
│   ├── Sales
│   │   ├── Sales-Users
│   │   ├── Sales-Computers
│   │   └── Sales-Groups
│   └── IT Department
│       ├── IT-Users
│       ├── IT-Computers
│       └── IT-Groups
├── 🏢 Servers
│   ├── Domain Controllers
│   ├── File Servers
│   ├── Application Servers
│   └── Web Servers
├── 🏢 Workstations
│   ├── Desktops
│   ├── Laptops
│   └── Kiosks
├── 🏢 Groups
│   ├── Security Groups
│   ├── Distribution Groups
│   └── Application Groups
└── 🏢 Special
    ├── Disabled Accounts
    ├── Temporary Accounts
    └── External Users
```

## 🔧 OU Creation Procedures

### Step 1: Create Top-Level OUs

#### 1.1 Administrative OU
```powershell
# Create Administrative OU
New-ADOrganizationalUnit -Name "Administrative" -Path "DC=rev,DC=local" -Description "Administrative accounts and resources"

# Create sub-OUs
New-ADOrganizationalUnit -Name "Admin-Accounts" -Path "OU=Administrative,DC=rev,DC=local" -Description "Domain and service administrator accounts"
New-ADOrganizationalUnit -Name "Service-Accounts" -Path "OU=Administrative,DC=rev,DC=local" -Description "Service and application accounts"
New-ADOrganizationalUnit -Name "Admin-Workstations" -Path "OU=Administrative,DC=rev,DC=local" -Description "Administrator workstations"
```

#### 1.2 Departments OU
```powershell
# Create Departments OU
New-ADOrganizationalUnit -Name "Departments" -Path "DC=rev,DC=local" -Description "Departmental organization"
```

#### 1.3 Servers OU
```powershell
# Create Servers OU
New-ADOrganizationalUnit -Name "Servers" -Path "DC=rev,DC=local" -Description "Server infrastructure"

# Create server sub-OUs
New-ADOrganizationalUnit -Name "Domain Controllers" -Path "OU=Servers,DC=rev,DC=local" -Description "Domain controller servers"
New-ADOrganizationalUnit -Name "File Servers" -Path "OU=Servers,DC=rev,DC=local" -Description "File and storage servers"
New-ADOrganizationalUnit -Name "Application Servers" -Path "OU=Servers,DC=rev,DC=local" -Description "Application and database servers"
New-ADOrganizationalUnit -Name "Web Servers" -Path "OU=Servers,DC=rev,DC=local" -Description "Web and application servers"
```

### Step 2: Create Departmental OUs

#### 2.1 Human Resources OU Structure
```powershell
# Create HR department OU
New-ADOrganizationalUnit -Name "Human Resources" -Path "OU=Departments,DC=rev,DC=local" -Description "Human Resources department"

# Create HR sub-OUs
New-ADOrganizationalUnit -Name "HR-Users" -Path "OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "HR department user accounts"
New-ADOrganizationalUnit -Name "HR-Computers" -Path "OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "HR department computers"
New-ADOrganizationalUnit -Name "HR-Groups" -Path "OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "HR department groups"
```

#### 2.2 Housekeeping OU Structure
```powershell
# Create Housekeeping department OU
New-ADOrganizationalUnit -Name "Housekeeping" -Path "OU=Departments,DC=rev,DC=local" -Description "Housekeeping department"

# Create Housekeeping sub-OUs
New-ADOrganizationalUnit -Name "HK-Users" -Path "OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Housekeeping department user accounts"
New-ADOrganizationalUnit -Name "HK-Computers" -Path "OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Housekeeping department computers"
New-ADOrganizationalUnit -Name "HK-Groups" -Path "OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Housekeeping department groups"
```

#### 2.3 Sales OU Structure
```powershell
# Create Sales department OU
New-ADOrganizationalUnit -Name "Sales" -Path "OU=Departments,DC=rev,DC=local" -Description "Sales department"

# Create Sales sub-OUs
New-ADOrganizationalUnit -Name "Sales-Users" -Path "OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Sales department user accounts"
New-ADOrganizationalUnit -Name "Sales-Computers" -Path "OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Sales department computers"
New-ADOrganizationalUnit -Name "Sales-Groups" -Path "OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Sales department groups"
```

#### 2.4 IT Department OU Structure
```powershell
# Create IT department OU
New-ADOrganizationalUnit -Name "IT Department" -Path "OU=Departments,DC=rev,DC=local" -Description "IT department"

# Create IT sub-OUs
New-ADOrganizationalUnit -Name "IT-Users" -Path "OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "IT department user accounts"
New-ADOrganizationalUnit -Name "IT-Computers" -Path "OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "IT department computers"
New-ADOrganizationalUnit -Name "IT-Groups" -Path "OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "IT department groups"
```

### Step 3: Create Supporting OUs

#### 3.1 Workstations OU
```powershell
# Create Workstations OU
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=rev,DC=local" -Description "All workstation computers"

# Create workstation sub-OUs
New-ADOrganizationalUnit -Name "Desktops" -Path "OU=Workstations,DC=rev,DC=local" -Description "Desktop computers"
New-ADOrganizationalUnit -Name "Laptops" -Path "OU=Workstations,DC=rev,DC=local" -Description "Laptop computers"
New-ADOrganizationalUnit -Name "Kiosks" -Path "OU=Workstations,DC=rev,DC=local" -Description "Kiosk and shared computers"
```

#### 3.2 Groups OU
```powershell
# Create Groups OU
New-ADOrganizationalUnit -Name "Groups" -Path "DC=rev,DC=local" -Description "Global groups"

# Create group sub-OUs
New-ADOrganizationalUnit -Name "Security Groups" -Path "OU=Groups,DC=rev,DC=local" -Description "Security groups for access control"
New-ADOrganizationalUnit -Name "Distribution Groups" -Path "OU=Groups,DC=rev,DC=local" -Description "Distribution groups for email"
New-ADOrganizationalUnit -Name "Application Groups" -Path "OU=Groups,DC=rev,DC=local" -Description "Application-specific groups"
```

#### 3.3 Special OU
```powershell
# Create Special OU
New-ADOrganizationalUnit -Name "Special" -Path "DC=rev,DC=local" -Description "Special purpose accounts"

# Create special sub-OUs
New-ADOrganizationalUnit -Name "Disabled Accounts" -Path "OU=Special,DC=rev,DC=local" -Description "Disabled user accounts"
New-ADOrganizationalUnit -Name "Temporary Accounts" -Path "OU=Special,DC=rev,DC=local" -Description "Temporary user accounts"
New-ADOrganizationalUnit -Name "External Users" -Path "OU=Special,DC=rev,DC=local" -Description "External and contractor accounts"
```

## 📋 OU Configuration

### OU Protection Settings
```powershell
# Enable protection from accidental deletion for critical OUs
$ous = @(
    "OU=Administrative,DC=rev,DC=local",
    "OU=Departments,DC=rev,DC=local",
    "OU=Servers,DC=rev,DC=local",
    "OU=Groups,DC=rev,DC=local"
)

foreach ($ou in $ous) {
    Get-ADOrganizationalUnit -Identity $ou | Set-ADOrganizationalUnit -ProtectedFromAccidentalDeletion $true
}
```

### OU Description Standards
| OU Type | Description Format | Example |
|---------|-------------------|---------|
| Department | "[Department Name] department" | "Human Resources department" |
| Server Type | "[Server Type] servers" | "Domain controller servers" |
| User Type | "[Department] department user accounts" | "HR department user accounts" |
| Computer Type | "[Department] department computers" | "HR department computers" |
| Group Type | "[Department] department groups" | "HR department groups" |

## 🔐 Delegation Configuration

### Departmental Delegation
```powershell
# Create departmental admin groups
New-ADGroup -Name "HR-Admins" -SamAccountName "HR-Admins" -GroupCategory Security -GroupScope Global -DisplayName "HR Administrators" -Path "OU=HR-Groups,OU=Human Resources,OU=Departments,DC=rev,DC=local"
New-ADGroup -Name "HK-Admins" -SamAccountName "HK-Admins" -GroupCategory Security -GroupScope Global -DisplayName "Housekeeping Administrators" -Path "OU=HK-Groups,OU=Housekeeping,OU=Departments,DC=rev,DC=local"
New-ADGroup -Name "Sales-Admins" -SamAccountName "Sales-Admins" -GroupCategory Security -GroupScope Global -DisplayName "Sales Administrators" -Path "OU=Sales-Groups,OU=Sales,OU=Departments,DC=rev,DC=local"
```

### Delegate Permissions
```powershell
# Delegate user management to HR-Admins
$ouPath = "OU=Human Resources,OU=Departments,DC=rev,DC=local"
$group = Get-ADGroup "HR-Admins"

# Grant user creation permissions
dsacls $ouPath /G "$($group.SID):CCDC;user"

# Grant password reset permissions
dsacls $ouPath /G "$($group.SID):CA;Reset Password;user"

# Grant group modification permissions
dsacls $ouPath /G "$($group.SID):CA;WriteProperty;group"
```

## 📊 OU Management Procedures

### OU Creation Approval Process
1. **Request Submission**: Submit OU creation request
2. **Impact Analysis**: Review impact on existing structure
3. **Design Review**: Validate OU design principles
4. **Approval**: Obtain management approval
5. **Implementation**: Create OU with proper settings
6. **Documentation**: Update OU documentation
7. **Communication**: Notify affected stakeholders

### OU Modification Process
1. **Change Request**: Submit modification request
2. **Risk Assessment**: Evaluate impact on Group Policy and delegation
3. **Testing**: Test changes in non-production environment
4. **Approval**: Obtain required approvals
5. **Implementation**: Apply changes during maintenance window
6. **Validation**: Verify Group Policy inheritance
7. **Documentation**: Update all relevant documentation

### OU Deletion Process
1. **Verification**: Confirm OU is empty
2. **Backup**: Export OU contents for backup
3. **Dependency Check**: Verify no Group Policies reference OU
4. **Approval**: Obtain management approval
5. **Deletion**: Remove OU from Active Directory
6. **Cleanup**: Clean up any remaining references
7. **Documentation**: Update OU structure documentation

## 🔍 OU Validation

### Structure Validation
```powershell
# Verify OU structure
Get-ADOrganizationalUnit -Filter * | Sort-Object DistinguishedName | Format-Table Name, DistinguishedName

# Check OU protection status
Get-ADOrganizationalUnit -Filter * | Select-Object Name, ProtectedFromAccidentalDeletion

# Verify OU descriptions
Get-ADOrganizationalUnit -Filter * | Select-Object Name, Description | Where-Object {$_.Description -eq $null}
```

### Delegation Validation
```powershell
# Check OU permissions
$ous = Get-ADOrganizationalUnit -Filter *
foreach ($ou in $ous) {
    Write-Host "Checking permissions for: $($ou.DistinguishedName)"
    dsacls $ou.DistinguishedName
}
```

### Group Policy Inheritance Validation
```powershell
# Check Group Policy inheritance
Get-GPInheritance -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local"
Get-GPInheritance -Target "OU=Sales,OU=Departments,DC=rev,DC=local"
Get-GPInheritance -Target "OU=IT Department,OU=Departments,DC=rev,DC=local"
```

## 🖼️ Screenshots

### OU Structure
![Active Directory OU Structure](../screenshots/02-active-directory/ou-structure.png)

### OU Creation
![OU Creation Wizard](../screenshots/02-active-directory/ou-creation.png)

### Delegation Configuration
![Delegation Wizard](../screenshots/02-active-directory/delegation-wizard.png)

### Group Policy Inheritance
![GP Inheritance](../screenshots/02-active-directory/gp-inheritance.png)

## 📋 OU Structure Summary

| OU Level | Count | Purpose |
|----------|-------|---------|
| Root Level | 6 | Main organizational categories |
| Department Level | 4 | Business departments |
| Sub-OU Level | 25 | Functional subdivisions |
| Total OUs | 35 | Complete organizational structure |

### Key Features Implemented
- ✅ Hierarchical departmental organization
- ✅ Separation of users, computers, and groups
- ✅ Administrative delegation capabilities
- ✅ Scalable structure for future growth
- ✅ Protection for critical OUs
- ✅ Clear naming conventions
- ✅ Comprehensive documentation

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
