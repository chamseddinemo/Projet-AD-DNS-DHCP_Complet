# User Account Management

## 📋 Description

This document provides comprehensive procedures for creating, configuring, and managing user accounts in the REV-Enterprise-Lab Active Directory environment, including user creation templates, departmental organization, and security best practices.

## 🎯 Objectives

- Create standardized user accounts for all departments
- Implement proper organizational unit placement
- Configure user account properties and security settings
- Establish user lifecycle management procedures
- Ensure compliance with naming conventions

## 👥 User Account Standards

### Naming Conventions
| Account Type | Format | Examples |
|--------------|--------|----------|
| Standard Users | firstname.lastname | john.smith, jane.doe |
| Service Accounts | svc-[application] | svc-sql, svc-backup |
| Administrative | admin-[function] | admin-it, admin-hr |
| Temporary | temp-[department]-[nn] | temp-hr-01, temp-it-01 |
| External | ext-[company]-[name] | ext-consulting-john |

### Password Standards
| Setting | Requirement |
|---------|-------------|
| Minimum Length | 6 characters |
| Complexity | Enabled (uppercase, lowercase, numbers, symbols) |
| Expiration | 60 days |
| History | Remember last 3 passwords |
| Account Lockout | 5 failed attempts, 30 minutes lockout |

## 🏢 Departmental User Creation

### Human Resources Users

#### HR Manager Account
```powershell
# Create HR Manager account
New-ADUser -Name "John Smith" -GivenName "John" -Surname "Smith" -SamAccountName "john.smith" -UserPrincipalName "john.smith@rev.local" -Path "OU=HR-Users,OU=Human Resources,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "Human Resources" -Title "HR Manager" -Office "Main Office" -OfficePhone "x1001" -EmailAddress "john.smith@rev.local" -Description "Human Resources Manager"

# Add to HR security group
Add-ADGroupMember -Identity "HR-Group" -Members "john.smith"
```

#### HR Specialist Account
```powershell
# Create HR Specialist account
New-ADUser -Name "Sarah Johnson" -GivenName "Sarah" -Surname "Johnson" -SamAccountName "sarah.johnson" -UserPrincipalName "sarah.johnson@rev.local" -Path "OU=HR-Users,OU=Human Resources,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "Human Resources" -Title "HR Specialist" -Office "Main Office" -OfficePhone "x1002" -EmailAddress "sarah.johnson@rev.local" -Description "Human Resources Specialist"

# Add to HR security group
Add-ADGroupMember -Identity "HR-Group" -Members "sarah.johnson"
```

### Housekeeping Users

#### Housekeeping Supervisor Account
```powershell
# Create Housekeeping Supervisor account
New-ADUser -Name "Michael Brown" -GivenName "Michael" -Surname "Brown" -SamAccountName "michael.brown" -UserPrincipalName "michael.brown@rev.local" -Path "OU=HK-Users,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "Housekeeping" -Title "Housekeeping Supervisor" -Office "Main Office" -OfficePhone "x2001" -EmailAddress "michael.brown@rev.local" -Description "Housekeeping Supervisor"

# Add to Housekeeping security group
Add-ADGroupMember -Identity "HK-Group" -Members "michael.brown"
```

#### Housekeeping Staff Account
```powershell
# Create Housekeeping Staff account
New-ADUser -Name "Emily Davis" -GivenName "Emily" -Surname "Davis" -SamAccountName "emily.davis" -UserPrincipalName "emily.davis@rev.local" -Path "OU=HK-Users,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "Housekeeping" -Title "Housekeeping Staff" -Office "Main Office" -OfficePhone "x2002" -EmailAddress "emily.davis@rev.local" -Description "Housekeeping Staff"

# Add to Housekeeping security group
Add-ADGroupMember -Identity "HK-Group" -Members "emily.davis"
```

### Sales Users

#### Sales Manager Account
```powershell
# Create Sales Manager account
New-ADUser -Name "Robert Wilson" -GivenName "Robert" -Surname "Wilson" -SamAccountName "robert.wilson" -UserPrincipalName "robert.wilson@rev.local" -Path "OU=Sales-Users,OU=Sales,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "Sales" -Title "Sales Manager" -Office "Main Office" -OfficePhone "x3001" -EmailAddress "robert.wilson@rev.local" -Description "Sales Manager"

# Add to Sales security group
Add-ADGroupMember -Identity "Sales-Group" -Members "robert.wilson"
```

#### Sales Representative Account
```powershell
# Create Sales Representative account
New-ADUser -Name "Jennifer Martinez" -GivenName "Jennifer" -Surname "Martinez" -SamAccountName "jennifer.martinez" -UserPrincipalName "jennifer.martinez@rev.local" -Path "OU=Sales-Users,OU=Sales,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "Sales" -Title "Sales Representative" -Office "Main Office" -OfficePhone "x3002" -EmailAddress "jennifer.martinez@rev.local" -Description "Sales Representative"

# Add to Sales security group
Add-ADGroupMember -Identity "Sales-Group" -Members "jennifer.martinez"
```

### IT Department Users

#### IT Administrator Account
```powershell
# Create IT Administrator account
New-ADUser -Name "David Lee" -GivenName "David" -Surname "Lee" -SamAccountName "david.lee" -UserPrincipalName "david.lee@rev.local" -Path "OU=IT-Users,OU=IT Department,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $false -Department "IT" -Title "IT Administrator" -Office "Main Office" -OfficePhone "x4001" -EmailAddress "david.lee@rev.local" -Description "IT Administrator"

# Add to IT security group
Add-ADGroupMember -Identity "IT-Group" -Members "david.lee"

# Add to Domain Admins (temporary for setup)
Add-ADGroupMember -Identity "Domain Admins" -Members "david.lee"
```

#### IT Support Account
```powershell
# Create IT Support account
New-ADUser -Name "Lisa Anderson" -GivenName "Lisa" -Surname "Anderson" -SamAccountName "lisa.anderson" -UserPrincipalName "lisa.anderson@rev.local" -Path "OU=IT-Users,OU=IT Department,OU=Departments,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department "IT" -Title "IT Support Specialist" -Office "Main Office" -OfficePhone "x4002" -EmailAddress "lisa.anderson@rev.local" -Description "IT Support Specialist"

# Add to IT security group
Add-ADGroupMember -Identity "IT-Group" -Members "lisa.anderson"
```

## 🔧 Service Account Creation

### SQL Service Account
```powershell
# Create SQL Service account
New-ADUser -Name "SQL Service Account" -GivenName "SQL" -Surname "Service" -SamAccountName "svc-sql" -UserPrincipalName "svc-sql@rev.local" -Path "OU=Service-Accounts,OU=Administrative,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "ComplexP@ssw0rd!2026" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $false -PasswordNeverExpires $true -Department "IT" -Title "SQL Service Account" -Description "SQL Server service account"

# Add to appropriate service groups
Add-ADGroupMember -Identity "Service Accounts" -Members "svc-sql"
```

### Backup Service Account
```powershell
# Create Backup Service account
New-ADUser -Name "Backup Service Account" -GivenName "Backup" -Surname "Service" -SamAccountName "svc-backup" -UserPrincipalName "svc-backup@rev.local" -Path "OU=Service-Accounts,OU=Administrative,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "ComplexP@ssw0rd!2026" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $false -PasswordNeverExpires $true -Department "IT" -Title "Backup Service Account" -Description "Backup service account"

# Add to appropriate service groups
Add-ADGroupMember -Identity "Service Accounts" -Members "svc-backup"
```

## 📋 User Account Templates

### Standard User Template
```powershell
# Function to create standard user
function New-StandardUser {
    param(
        [string]$FirstName,
        [string]$LastName,
        [string]$Department,
        [string]$Title,
        [string]$Office,
        [string]$Phone,
        [string]$OUPath
    )
    
    $samAccount = "$($FirstName.ToLower()).$($LastName.ToLower())"
    $userPrincipal = "$samAccount@rev.local"
    $displayName = "$FirstName $LastName"
    $email = "$samAccount@rev.local"
    $description = "$Title - $Department"
    
    New-ADUser -Name $displayName -GivenName $FirstName -Surname $LastName -SamAccountName $samAccount -UserPrincipalName $userPrincipal -Path $OUPath -AccountPassword (ConvertTo-SecureString "TempP@ss123" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department $Department -Title $Title -Office $Office -OfficePhone $Phone -EmailAddress $email -Description $description
    
    Write-Host "Created user: $samAccount"
}

# Example usage
New-StandardUser -FirstName "Test" -LastName "User" -Department "IT" -Title "Test User" -Office "Main Office" -Phone "x9999" -OUPath "OU=IT-Users,OU=IT Department,OU=Departments,DC=rev,DC=local"
```

### Bulk User Creation
```powershell
# Import users from CSV
$users = Import-Csv -Path "C:\Temp\NewUsers.csv"

foreach ($user in $users) {
    $ouPath = switch ($user.Department) {
        "Human Resources" { "OU=HR-Users,OU=Human Resources,OU=Departments,DC=rev,DC=local" }
        "Housekeeping" { "OU=HK-Users,OU=Housekeeping,OU=Departments,DC=rev,DC=local" }
        "Sales" { "OU=Sales-Users,OU=Sales,OU=Departments,DC=rev,DC=local" }
        "IT" { "OU=IT-Users,OU=IT Department,OU=Departments,DC=rev,DC=local" }
        default { "CN=Users,DC=rev,DC=local" }
    }
    
    New-ADUser -Name $user.DisplayName -GivenName $user.FirstName -Surname $user.LastName -SamAccountName $user.SamAccountName -UserPrincipalName $user.UserPrincipalName -Path $ouPath -AccountPassword (ConvertTo-SecureString $user.Password -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $true -Department $user.Department -Title $user.Title -Office $user.Office -OfficePhone $user.Phone -EmailAddress $user.Email -Description $user.Description
    
    # Add to department group
    $groupName = switch ($user.Department) {
        "Human Resources" { "HR-Group" }
        "Housekeeping" { "HK-Group" }
        "Sales" { "Sales-Group" }
        "IT" { "IT-Group" }
        default { "Domain Users" }
    }
    
    Add-ADGroupMember -Identity $groupName -Members $user.SamAccountName
}
```

## 🔄 User Account Management

### User Account Modification
```powershell
# Modify user properties
Set-ADUser -Identity "john.smith" -Department "Human Resources" -Title "Senior HR Manager" -Office "Main Office" -OfficePhone "x1001" -EmailAddress "john.smith@rev.local" -Description "Senior Human Resources Manager"

# Update user password
Set-ADAccountPassword -Identity "john.smith" -Reset -NewPassword (ConvertTo-SecureString "NewP@ssw0rd123" -AsPlainText -Force)

# Enable/disable user account
Enable-ADAccount -Identity "john.smith"
Disable-ADAccount -Identity "john.smith"
```

### User Account Transfer
```powershell
# Transfer user between departments
$user = Get-ADUser -Identity "jennifer.martinez"
$targetOU = "OU=HR-Users,OU=Human Resources,OU=Departments,DC=rev,DC=local"

Move-ADObject -Identity $user.DistinguishedName -TargetPath $targetOU

# Update user properties
Set-ADUser -Identity "jennifer.martinez" -Department "Human Resources" -Title "HR Coordinator" -Description "Human Resources Coordinator"

# Remove from old group and add to new group
Remove-ADGroupMember -Identity "Sales-Group" -Members "jennifer.martinez" -Confirm:$false
Add-ADGroupMember -Identity "HR-Group" -Members "jennifer.martinez"
```

## 📊 User Account Reporting

### User Inventory Report
```powershell
# Generate user inventory report
$users = Get-ADUser -Filter * -Properties Department, Title, Office, OfficePhone, EmailAddress, LastLogonDate, PasswordLastSet

$report = $users | Select-Object Name, SamAccountName, Department, Title, Office, OfficePhone, EmailAddress, Enabled, LastLogonDate, PasswordLastSet | Sort-Object Department, Name

$report | Export-Csv -Path "C:\Reports\UserInventory.csv" -NoTypeInformation
$report | Out-GridView
```

### Departmental User Count
```powershell
# Count users by department
$users = Get-ADUser -Filter * -Properties Department
$departmentCounts = $users | Group-Object Department | Sort-Object Name

foreach ($dept in $departmentCounts) {
    Write-Host "$($dept.Name): $($dept.Count) users"
}
```

### Inactive User Accounts
```powershell
# Find inactive accounts (90+ days)
$inactiveDate = (Get-Date).AddDays(-90)
$inactiveUsers = Get-ADUser -Filter {LastLogonDate -lt $inactiveDate -and Enabled -eq $true} -Properties LastLogonDate, Description

$inactiveUsers | Select-Object Name, SamAccountName, LastLogonDate, Description | Export-Csv -Path "C:\Reports\InactiveUsers.csv" -NoTypeInformation
```

## 🔒 Security Best Practices

### Account Security Settings
```powershell
# Configure user account security
Set-ADAccountControl -Identity "john.smith" -DoesNotRequirePreAuth $false -PasswordNeverExpires $false -TrustedForDelegation $false -AllowReversiblePasswordEncryption $false

# Set user logon restrictions
Set-ADUser -Identity "john.smith" -LogonWorkstations "HRPC01,HRPC02" -LogonHours (New-Object byte[] 21)

# Configure user profile settings
Set-ADUser -Identity "john.smith" -ProfilePath "\\PDC\Profiles$\john.smith" -HomeDirectory "\\PDC\Home$\john.smith" -HomeDrive "H:"
```

### Privileged Account Management
```powershell
# Create admin account with separate credentials
New-ADUser -Name "Admin David Lee" -GivenName "Admin" -Surname "Lee" -SamAccountName "admin.david" -UserPrincipalName "admin.david@rev.local" -Path "OU=Admin-Accounts,OU=Administrative,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "AdminP@ssw0rd!2026" -AsPlainText -Force) -Enabled $true -ChangePasswordAtLogon $false -PasswordNeverExpires $true -Department "IT" -Title "Administrator" -Description "Administrative account for David Lee"

# Add to administrative groups
Add-ADGroupMember -Identity "Domain Admins" -Members "admin.david"
Add-ADGroupMember -Identity "Enterprise Admins" -Members "admin.david"
```

## 🖼️ Screenshots

### User Creation
![New User Creation](../screenshots/02-active-directory/user-creation.png)

### User Properties
![User Properties Dialog](../screenshots/02-active-directory/user-properties.png)

### User Management
![Active Directory Users](../screenshots/02-active-directory/user-management.png)

### Departmental Organization
![Departmental Users](../screenshots/02-active-directory/departmental-users.png)

## 📋 User Account Summary

| Department | Users Created | Groups | Status |
|------------|----------------|--------|--------|
| Human Resources | 2 | HR-Group | ✅ Complete |
| Housekeeping | 2 | HK-Group | ✅ Complete |
| Sales | 2 | Sales-Group | ✅ Complete |
| IT Department | 2 | IT-Group | ✅ Complete |
| Service Accounts | 2 | Service Accounts | ✅ Complete |

### Total User Accounts Created: 10

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
