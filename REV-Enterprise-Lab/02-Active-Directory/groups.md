# Group Management

## 📋 Description

This document provides comprehensive procedures for creating and managing Active Directory groups in the REV-Enterprise-Lab environment, including security groups, distribution groups, and group nesting strategies for efficient access control.

## 🎯 Objectives

- Create departmental security groups for access control
- Implement distribution groups for email communication
- Establish group nesting strategies for simplified management
- Configure group scopes and types appropriately
- Document group membership and permissions

## 👥 Group Types and Scopes

### Group Types
| Type | Purpose | Usage |
|------|---------|-------|
| Security | Access control | File permissions, resource access |
| Distribution | Email communication | Email lists, notifications |
| Mail-Enabled Security | Both access and email | Resource access with email functionality |

### Group Scopes
| Scope | Domain | Forest | Universal | Usage |
|-------|--------|--------|-----------|-------|
| Domain Local | ✅ | ❌ | ❌ | Resource access within domain |
| Global | ✅ | ❌ | ❌ | User organization, can be nested |
| Universal | ✅ | ✅ | ✅ | Cross-domain access, email distribution |

## 🏢 Departmental Security Groups

### Human Resources Groups

#### HR-Group (Global Security)
```powershell
# Create HR main security group
New-ADGroup -Name "HR-Group" -SamAccountName "HR-Group" -GroupCategory Security -GroupScope Global -DisplayName "Human Resources Group" -Path "OU=HR-Groups,OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "Main security group for Human Resources department members"

# Add HR users to the group
Add-ADGroupMember -Identity "HR-Group" -Members "john.smith", "sarah.johnson"
```

#### HR-Admins (Global Security)
```powershell
# Create HR administrators group
New-ADGroup -Name "HR-Admins" -SamAccountName "HR-Admins" -GroupCategory Security -GroupScope Global -DisplayName "HR Administrators" -Path "OU=HR-Groups,OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "Administrative group for Human Resources department"

# Add HR manager to admins group
Add-ADGroupMember -Identity "HR-Admins" -Members "john.smith"
```

#### HR-File-Access (Domain Local Security)
```powershell
# Create HR file access group
New-ADGroup -Name "HR-File-Access" -SamAccountName "HR-File-Access" -GroupCategory Security -GroupScope DomainLocal -DisplayName "HR File Access" -Path "OU=HR-Groups,OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "Access to HR shared folders and files"

# Add HR-Group as member (group nesting)
Add-ADGroupMember -Identity "HR-File-Access" -Members "HR-Group"
```

### Housekeeping Groups

#### HK-Group (Global Security)
```powershell
# Create Housekeeping main security group
New-ADGroup -Name "HK-Group" -SamAccountName "HK-Group" -GroupCategory Security -GroupScope Global -DisplayName "Housekeeping Group" -Path "OU=HK-Groups,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Main security group for Housekeeping department members"

# Add Housekeeping users to the group
Add-ADGroupMember -Identity "HK-Group" -Members "michael.brown", "emily.davis"
```

#### HK-Admins (Global Security)
```powershell
# Create Housekeeping administrators group
New-ADGroup -Name "HK-Admins" -SamAccountName "HK-Admins" -GroupCategory Security -GroupScope Global -DisplayName "Housekeeping Administrators" -Path "OU=HK-Groups,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Administrative group for Housekeeping department"

# Add Housekeeping supervisor to admins group
Add-ADGroupMember -Identity "HK-Admins" -Members "michael.brown"
```

#### HK-Schedule-Access (Domain Local Security)
```powershell
# Create Housekeeping schedule access group
New-ADGroup -Name "HK-Schedule-Access" -SamAccountName "HK-Schedule-Access" -GroupCategory Security -GroupScope DomainLocal -DisplayName "Housekeeping Schedule Access" -Path "OU=HK-Groups,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Access to housekeeping scheduling system"

# Add HK-Group as member
Add-ADGroupMember -Identity "HK-Schedule-Access" -Members "HK-Group"
```

### Sales Groups

#### Sales-Group (Global Security)
```powershell
# Create Sales main security group
New-ADGroup -Name "Sales-Group" -SamAccountName "Sales-Group" -GroupCategory Security -GroupScope Global -DisplayName "Sales Group" -Path "OU=Sales-Groups,OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Main security group for Sales department members"

# Add Sales users to the group
Add-ADGroupMember -Identity "Sales-Group" -Members "robert.wilson", "jennifer.martinez"
```

#### Sales-Admins (Global Security)
```powershell
# Create Sales administrators group
New-ADGroup -Name "Sales-Admins" -SamAccountName "Sales-Admins" -GroupCategory Security -GroupScope Global -DisplayName "Sales Administrators" -Path "OU=Sales-Groups,OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Administrative group for Sales department"

# Add Sales manager to admins group
Add-ADGroupMember -Identity "Sales-Admins" -Members "robert.wilson"
```

#### Sales-CRM-Access (Domain Local Security)
```powershell
# Create Sales CRM access group
New-ADGroup -Name "Sales-CRM-Access" -SamAccountName "Sales-CRM-Access" -GroupCategory Security -GroupScope DomainLocal -DisplayName "Sales CRM Access" -Path "OU=Sales-Groups,OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Access to Customer Relationship Management system"

# Add Sales-Group as member
Add-ADGroupMember -Identity "Sales-CRM-Access" -Members "Sales-Group"
```

### IT Department Groups

#### IT-Group (Global Security)
```powershell
# Create IT main security group
New-ADGroup -Name "IT-Group" -SamAccountName "IT-Group" -GroupCategory Security -GroupScope Global -DisplayName "IT Group" -Path "OU=IT-Groups,OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "Main security group for IT department members"

# Add IT users to the group
Add-ADGroupMember -Identity "IT-Group" -Members "david.lee", "lisa.anderson"
```

#### IT-Admins (Global Security)
```powershell
# Create IT administrators group
New-ADGroup -Name "IT-Admins" -SamAccountName "IT-Admins" -GroupCategory Security -GroupScope Global -DisplayName "IT Administrators" -Path "OU=IT-Groups,OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "Administrative group for IT department"

# Add IT administrator to admins group
Add-ADGroupMember -Identity "IT-Admins" -Members "david.lee"
```

#### IT-Server-Admins (Global Security)
```powershell
# Create IT server administrators group
New-ADGroup -Name "IT-Server-Admins" -SamAccountName "IT-Server-Admins" -GroupCategory Security -GroupScope Global -DisplayName "IT Server Administrators" -Path "OU=IT-Groups,OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "Server administration rights for IT staff"

# Add IT-Admins as member
Add-ADGroupMember -Identity "IT-Server-Admins" -Members "IT-Admins"
```

## 📧 Distribution Groups

### All Employees (Universal Distribution)
```powershell
# Create all employees distribution group
New-ADGroup -Name "All Employees" -SamAccountName "All-Employees" -GroupCategory Distribution -GroupScope Universal -DisplayName "All Employees" -Path "OU=Distribution Groups,OU=Groups,DC=rev,DC=local" -Description "All company employees for company-wide communications"

# Add all department groups
Add-ADGroupMember -Identity "All Employees" -Members "HR-Group", "HK-Group", "Sales-Group", "IT-Group"
```

### Management Team (Universal Distribution)
```powershell
# Create management distribution group
New-ADGroup -Name "Management Team" -SamAccountName "Management-Team" -GroupCategory Distribution -GroupScope Universal -DisplayName "Management Team" -Path "OU=Distribution Groups,OU=Groups,DC=rev,DC=local" -Description "Management team for executive communications"

# Add management members
Add-ADGroupMember -Identity "Management Team" -Members "john.smith", "michael.brown", "robert.wilson", "david.lee"
```

### Department Announcements (Universal Distribution)
```powershell
# Create department announcement groups
New-ADGroup -Name "HR-Announcements" -SamAccountName "HR-Announcements" -GroupCategory Distribution -GroupScope Universal -DisplayName "HR Announcements" -Path "OU=Distribution Groups,OU=Groups,DC=rev,DC=local" -Description "HR department announcements and notifications"

New-ADGroup -Name "Sales-Announcements" -SamAccountName "Sales-Announcements" -GroupCategory Distribution -GroupScope Universal -DisplayName "Sales Announcements" -Path "OU=Distribution Groups,OU=Groups,DC=rev,DC=local" -Description "Sales department announcements and notifications"

New-ADGroup -Name "IT-Announcements" -SamAccountName "IT-Announcements" -GroupCategory Distribution -GroupScope Universal -DisplayName "IT Announcements" -Path "OU=Distribution Groups,OU=Groups,DC=rev,DC=local" -Description "IT department announcements and notifications"

# Add respective department groups
Add-ADGroupMember -Identity "HR-Announcements" -Members "HR-Group"
Add-ADGroupMember -Identity "Sales-Announcements" -Members "Sales-Group"
Add-ADGroupMember -Identity "IT-Announcements" -Members "IT-Group"
```

## 🔧 Resource-Based Groups

### File Server Access Groups

#### Public-Access (Domain Local Security)
```powershell
# Create public file access group
New-ADGroup -Name "Public-Access" -SamAccountName "Public-Access" -GroupCategory Security -GroupScope DomainLocal -DisplayName "Public Access" -Path "OU=Security Groups,OU=Groups,DC=rev,DC=local" -Description "Access to public shared folders"

# Add all authenticated users
Add-ADGroupMember -Identity "Public-Access" -Members "Authenticated Users"
```

#### HR-Share-Access (Domain Local Security)
```powershell
# Create HR share access group
New-ADGroup -Name "HR-Share-Access" -SamAccountName "HR-Share-Access" -GroupCategory Security -GroupScope DomainLocal -DisplayName "HR Share Access" -Path "OU=Security Groups,OU=Groups,DC=rev,DC=local" -Description "Access to HR department shared folders"

# Add HR-Group as member
Add-ADGroupMember -Identity "HR-Share-Access" -Members "HR-Group"
```

### Application Access Groups

#### HR-App-Users (Domain Local Security)
```powershell
# Create HR application users group
New-ADGroup -Name "HR-App-Users" -SamAccountName "HR-App-Users" -GroupCategory Security -GroupScope DomainLocal -DisplayName "HR Application Users" -Path "OU=Application Groups,OU=Groups,DC=rev,DC=local" -Description "Access to HR management applications"

# Add HR-Group as member
Add-ADGroupMember -Identity "HR-App-Users" -Members "HR-Group"
```

#### Sales-CRM-Users (Domain Local Security)
```powershell
# Create Sales CRM users group
New-ADGroup -Name "Sales-CRM-Users" -SamAccountName "Sales-CRM-Users" -GroupCategory Security -GroupScope DomainLocal -DisplayName "Sales CRM Users" -Path "OU=Application Groups,OU=Groups,DC=rev,DC=local" -Description "Access to Sales CRM system"

# Add Sales-Group as member
Add-ADGroupMember -Identity "Sales-CRM-Users" -Members "Sales-Group"
```

## 🔄 Group Management Procedures

### Group Creation Template
```powershell
# Function to create standardized groups
function New-StandardGroup {
    param(
        [string]$Name,
        [string]$SamAccountName,
        [string]$DisplayName,
        [string]$Description,
        [string]$Category,
        [string]$Scope,
        [string]$OUPath
    )
    
    New-ADGroup -Name $Name -SamAccountName $SamAccountName -GroupCategory $Category -GroupScope $Scope -DisplayName $DisplayName -Path $OUPath -Description $Description
    
    Write-Host "Created group: $Name"
}

# Example usage
New-StandardGroup -Name "Test-Group" -SamAccountName "Test-Group" -DisplayName "Test Group" -Description "Test group for demonstration" -Category "Security" -Scope "Global" -OUPath "OU=Security Groups,OU=Groups,DC=rev,DC=local"
```

### Bulk Group Creation
```powershell
# Import groups from CSV
$groups = Import-Csv -Path "C:\Temp\NewGroups.csv"

foreach ($group in $groups) {
    New-ADGroup -Name $group.Name -SamAccountName $group.SamAccountName -GroupCategory $group.Category -GroupScope $group.Scope -DisplayName $group.DisplayName -Path $group.OUPath -Description $group.Description
    
    # Add members if specified
    if ($group.Members) {
        $members = $group.Members -split ","
        Add-ADGroupMember -Identity $group.SamAccountName -Members $members
    }
}
```

### Group Membership Management
```powershell
# Add multiple users to group
$users = @("user1", "user2", "user3")
Add-ADGroupMember -Identity "HR-Group" -Members $users

# Remove user from group
Remove-ADGroupMember -Identity "HR-Group" -Members "user1" -Confirm:$false

# Move user between groups
Remove-ADGroupMember -Identity "HR-Group" -Members "user1" -Confirm:$false
Add-ADGroupMember -Identity "Sales-Group" -Members "user1"
```

## 📊 Group Reporting

### Group Inventory Report
```powershell
# Generate comprehensive group report
$groups = Get-ADGroup -Filter * -Properties Description, GroupCategory, GroupScope, DistinguishedName, Created, Modified

$report = $groups | Select-Object Name, SamAccountName, GroupCategory, GroupScope, Description, DistinguishedName, Created, Modified | Sort-Object GroupCategory, Name

$report | Export-Csv -Path "C:\Reports\GroupInventory.csv" -NoTypeInformation
$report | Out-GridView
```

### Group Membership Report
```powershell
# Report on group memberships
$groups = Get-ADGroup -Filter * -Properties Members

foreach ($group in $groups) {
    $members = Get-ADGroupMember -Identity $group.DistinguishedName
    Write-Host "Group: $($group.Name) - Member Count: $($members.Count)"
    
    foreach ($member in $members) {
        Write-Host "  - $($member.Name) ($($member.objectClass))"
    }
    Write-Host ""
}
```

### Departmental Group Summary
```powershell
# Departmental group summary
$departments = @("HR", "HK", "Sales", "IT")

foreach ($dept in $departments) {
    $deptGroups = Get-ADGroup -Filter "Name -like '$dept*'" -Properties Description
    Write-Host "=== $dept Department Groups ==="
    
    foreach ($group in $deptGroups) {
        $memberCount = (Get-ADGroupMember -Identity $group.DistinguishedName).Count
        Write-Host "$($group.Name): $memberCount members"
    }
    Write-Host ""
}
```

## 🔒 Security Best Practices

### Group Naming Conventions
| Group Type | Naming Pattern | Examples |
|------------|----------------|----------|
| Department Security | [DEPT]-Group | HR-Group, IT-Group |
| Department Admins | [DEPT]-Admins | HR-Admins, IT-Admins |
| Resource Access | [RESOURCE]-Access | HR-Share-Access, Public-Access |
| Application Access | [APP]-Users | HR-App-Users, Sales-CRM-Users |
| Distribution | [Purpose]-Announcements | HR-Announcements, IT-Announcements |

### Group Nesting Strategy
```
Department Users (Global Groups)
    ↓
Resource Access Groups (Domain Local Groups)
    ↓
Resource Permissions
```

### Regular Maintenance Tasks
```powershell
# Find empty groups
$emptyGroups = Get-ADGroup -Filter * -Properties Members | Where-Object {$_.Members.Count -eq 0}
$emptyGroups | Select-Object Name, DistinguishedName

# Find groups with expired members
$allUsers = Get-ADUser -Filter * -Properties LastLogonDate
$disabledUsers = $allUsers | Where-Object {$_.Enabled -eq $false}

foreach ($user in $disabledUsers) {
    $groups = Get-ADGroup -Filter "Members -eq '$($user.DistinguishedName)'" -Properties Members
    foreach ($group in $groups) {
        Write-Host "Disabled user $($user.Name) is member of group $($group.Name)"
    }
}
```

## 🖼️ Screenshots

### Group Creation
![New Group Creation](../screenshots/02-active-directory/group-creation.png)

### Group Properties
![Group Properties Dialog](../screenshots/02-active-directory/group-properties.png)

### Group Membership
![Group Members](../screenshots/02-active-directory/group-membership.png)

### Group Nesting
![Group Nesting Structure](../screenshots/02-active-directory/group-nesting.png)

## 📋 Group Summary

| Category | Groups Created | Purpose |
|----------|----------------|---------|
| Department Security | 4 | Departmental access control |
| Department Admins | 4 | Administrative delegation |
| Resource Access | 6 | File and application access |
| Distribution | 4 | Email communication |
| Total Groups | 18 | Complete group structure |

### Key Features Implemented
- ✅ Departmental security groups
- ✅ Administrative delegation groups
- ✅ Resource-based access groups
- ✅ Distribution groups for communication
- ✅ Proper group nesting strategy
- ✅ Consistent naming conventions
- ✅ Comprehensive documentation

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
