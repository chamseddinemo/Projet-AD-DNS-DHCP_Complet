# Computer Account Management

## 📋 Description

This document provides comprehensive procedures for creating and managing computer accounts in the REV-Enterprise-Lab Active Directory environment, including workstation and server computer objects, naming conventions, and organizational unit placement.

## 🎯 Objectives

- Create computer accounts for all servers and workstations
- Implement proper naming conventions for computers
- Configure computer account properties and security settings
- Establish computer lifecycle management procedures
- Ensure proper OU placement for Group Policy application

## 💻 Computer Account Standards

### Naming Conventions
| Computer Type | Format | Examples |
|---------------|--------|----------|
| Domain Controllers | DC-[NN] | DC-01, DC-02 |
| File Servers | FS-[NN] | FS-01, FS-02 |
| Application Servers | APP-[NN] | APP-01, APP-02 |
| Web Servers | WEB-[NN] | WEB-01, WEB-02 |
| Workstations | [DEPT]-PC-[NN] | HR-PC-01, IT-PC-01 |
| Laptops | [DEPT]-LT-[NN] | HR-LT-01, IT-LT-01 |
| Virtual Machines | VM-[PURPOSE]-[NN] | VM-TEST-01, VM-DEV-01 |

### Computer Account Properties
| Property | Requirement | Description |
|----------|-------------|-------------|
| Description | Required | Department, user, and purpose |
| Location | Optional | Physical location of device |
| Managed By | Optional | Primary user or department |
| Operating System | Auto-detected | Version and service pack |
| Join Date | Auto-recorded | Domain join timestamp |

## 🖥️ Server Computer Accounts

### Primary Domain Controller
```powershell
# Verify PDC computer account
$dc = Get-ADComputer -Identity "PDC"
Set-ADComputer -Identity $dc -Description "Primary Domain Controller - Windows Server 2019" -ManagedBy "david.lee" -Location "Server Room"

# Add to Domain Controllers OU (should already be there)
Move-ADObject -Identity $dc.DistinguishedName -TargetPath "OU=Domain Controllers,OU=Servers,DC=rev,DC=local"
```

### File Server Account
```powershell
# Create file server computer account
New-ADComputer -Name "FS01" -SamAccountName "FS01$" -DNSHostName "FS01.rev.local" -Path "OU=File Servers,OU=Servers,DC=rev,DC=local" -Description "Primary File Server - Windows Server 2019" -ManagedBy "david.lee" -Location "Server Room" -Enabled $true

# Configure additional properties
Set-ADComputer -Identity "FS01" -OperatingSystem "Windows Server 2019" -OperatingSystemVersion "10.0 (17763)" -OperatingSystemServicePack "Latest"
```

### Application Server Account
```powershell
# Create application server computer account
New-ADComputer -Name "APP01" -SamAccountName "APP01$" -DNSHostName "APP01.rev.local" -Path "OU=Application Servers,OU=Servers,DC=rev,DC=local" -Description "Application Server - Windows Server 2019" -ManagedBy "david.lee" -Location "Server Room" -Enabled $true

# Configure properties
Set-ADComputer -Identity "APP01" -OperatingSystem "Windows Server 2019" -OperatingSystemVersion "10.0 (17763)" -OperatingSystemServicePack "Latest"
```

### Web Server Accounts
```powershell
# Create web server computer accounts
New-ADComputer -Name "WEB01" -SamAccountName "WEB01$" -DNSHostName "WEB01.rev.local" -Path "OU=Web Servers,OU=Servers,DC=rev,DC=local" -Description "Web Server 1 - Windows Server 2019" -ManagedBy "david.lee" -Location "Server Room" -Enabled $true

New-ADComputer -Name "WEB02" -SamAccountName "WEB02$" -DNSHostName "WEB02.rev.local" -Path "OU=Web Servers,OU=Servers,DC=rev,DC=local" -Description "Web Server 2 - Windows Server 2019" -ManagedBy "david.lee" -Location "Server Room" -Enabled $true

# Configure properties
Set-ADComputer -Identity "WEB01" -OperatingSystem "Windows Server 2019" -OperatingSystemVersion "10.0 (17763)" -OperatingSystemServicePack "Latest"
Set-ADComputer -Identity "WEB02" -OperatingSystem "Windows Server 2019" -OperatingSystemVersion "10.0 (17763)" -OperatingSystemServicePack "Latest"
```

## 💻 Workstation Computer Accounts

### Human Resources Workstations

#### HR Manager Workstation
```powershell
# Create HR manager workstation
New-ADComputer -Name "HR-PC-01" -SamAccountName "HR-PC-01$" -DNSHostName "HR-PC-01.rev.local" -Path "OU=HR-Computers,OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "HR Manager Workstation - Windows 10 Enterprise" -ManagedBy "john.smith" -Location "Main Office - HR Department" -Enabled $true

# Configure properties
Set-ADComputer -Identity "HR-PC-01" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"

# Set DHCP reservation (if needed)
# This would be done in DHCP management console
```

#### HR Specialist Workstation
```powershell
# Create HR specialist workstation
New-ADComputer -Name "HR-PC-02" -SamAccountName "HR-PC-02$" -DNSHostName "HR-PC-02.rev.local" -Path "OU=HR-Computers,OU=Human Resources,OU=Departments,DC=rev,DC=local" -Description "HR Specialist Workstation - Windows 10 Enterprise" -ManagedBy "sarah.johnson" -Location "Main Office - HR Department" -Enabled $true

# Configure properties
Set-ADComputer -Identity "HR-PC-02" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"
```

### Housekeeping Workstations

#### Housekeeping Supervisor Workstation
```powershell
# Create Housekeeping supervisor workstation
New-ADComputer -Name "HK-PC-01" -SamAccountName "HK-PC-01$" -DNSHostName "HK-PC-01.rev.local" -Path "OU=HK-Computers,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Housekeeping Supervisor Workstation - Windows 10 Enterprise" -ManagedBy "michael.brown" -Location "Main Office - Housekeeping" -Enabled $true

# Configure properties
Set-ADComputer -Identity "HK-PC-01" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"
```

#### Housekeeping Staff Workstation
```powershell
# Create Housekeeping staff workstation
New-ADComputer -Name "HK-PC-02" -SamAccountName "HK-PC-02$" -DNSHostName "HK-PC-02.rev.local" -Path "OU=HK-Computers,OU=Housekeeping,OU=Departments,DC=rev,DC=local" -Description "Housekeeping Staff Workstation - Windows 10 Enterprise" -ManagedBy "emily.davis" -Location "Main Office - Housekeeping" -Enabled $true

# Configure properties
Set-ADComputer -Identity "HK-PC-02" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"
```

### Sales Workstations

#### Sales Manager Workstation
```powershell
# Create Sales manager workstation
New-ADComputer -Name "SALES-PC-01" -SamAccountName "SALES-PC-01$" -DNSHostName "SALES-PC-01.rev.local" -Path "OU=Sales-Computers,OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Sales Manager Workstation - Windows 10 Enterprise" -ManagedBy "robert.wilson" -Location "Main Office - Sales Department" -Enabled $true

# Configure properties
Set-ADComputer -Identity "SALES-PC-01" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"
```

#### Sales Representative Workstation
```powershell
# Create Sales representative workstation
New-ADComputer -Name "SALES-PC-02" -SamAccountName "SALES-PC-02$" -DNSHostName "SALES-PC-02.rev.local" -Path "OU=Sales-Computers,OU=Sales,OU=Departments,DC=rev,DC=local" -Description "Sales Representative Workstation - Windows 10 Enterprise" -ManagedBy "jennifer.martinez" -Location "Main Office - Sales Department" -Enabled $true

# Configure properties
Set-ADComputer -Identity "SALES-PC-02" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"
```

### IT Department Workstations

#### IT Administrator Workstation
```powershell
# Create IT administrator workstation
New-ADComputer -Name "IT-PC-01" -SamAccountName "IT-PC-01$" -DNSHostName "IT-PC-01.rev.local" -Path "OU=IT-Computers,OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "IT Administrator Workstation - Windows 10 Enterprise" -ManagedBy "david.lee" -Location "Main Office - IT Department" -Enabled $true

# Configure properties
Set-ADComputer -Identity "IT-PC-01" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"

# Add to admin workstation group
Add-ADGroupMember -Identity "Admin-Workstations" -Members "IT-PC-01$"
```

#### IT Support Workstation
```powershell
# Create IT support workstation
New-ADComputer -Name "IT-PC-02" -SamAccountName "IT-PC-02$" -DNSHostName "IT-PC-02.rev.local" -Path "OU=IT-Computers,OU=IT Department,OU=Departments,DC=rev,DC=local" -Description "IT Support Workstation - Windows 10 Enterprise" -ManagedBy "lisa.anderson" -Location "Main Office - IT Department" -Enabled $true

# Configure properties
Set-ADComputer -Identity "IT-PC-02" -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"

# Add to admin workstation group
Add-ADGroupMember -Identity "Admin-Workstations" -Members "IT-PC-02$"
```

## 🔧 Computer Account Templates

### Standard Workstation Template
```powershell
# Function to create standard workstation
function New-StandardWorkstation {
    param(
        [string]$ComputerName,
        [string]$User,
        [string]$Department,
        [string]$Location,
        [string]$OUPath
    )
    
    $description = "$Department Workstation - Windows 10 Enterprise"
    $dnsHostName = "$ComputerName.rev.local"
    
    New-ADComputer -Name $ComputerName -SamAccountName "$ComputerName$" -DNSHostName $dnsHostName -Path $OUPath -Description $description -ManagedBy $User -Location $Location -Enabled $true
    
    # Configure standard properties
    Set-ADComputer -Identity $ComputerName -OperatingSystem "Windows 10 Enterprise" -OperatingSystemVersion "10.0 (19045)" -OperatingSystemServicePack "Latest"
    
    Write-Host "Created workstation: $ComputerName"
}

# Example usage
New-StandardWorkstation -ComputerName "TEST-PC-01" -User "test.user" -Department "IT" -Location "Test Lab" -OUPath "OU=IT-Computers,OU=IT Department,OU=Departments,DC=rev,DC=local"
```

### Standard Server Template
```powershell
# Function to create standard server
function New-StandardServer {
    param(
        [string]$ServerName,
        [string]$ServerType,
        [string]$Admin,
        [string]$Location,
        [string]$OUPath
    )
    
    $description = "$ServerType - Windows Server 2019"
    $dnsHostName = "$ServerName.rev.local"
    
    New-ADComputer -Name $ServerName -SamAccountName "$ServerName$" -DNSHostName $dnsHostName -Path $OUPath -Description $description -ManagedBy $Admin -Location $Location -Enabled $true
    
    # Configure standard properties
    Set-ADComputer -Identity $ServerName -OperatingSystem "Windows Server 2019" -OperatingSystemVersion "10.0 (17763)" -OperatingSystemServicePack "Latest"
    
    Write-Host "Created server: $ServerName"
}

# Example usage
New-StandardServer -ServerName "TEST-SRV-01" -ServerType "Test Server" -Admin "david.lee" -Location "Server Room" -OUPath "OU=Application Servers,OU=Servers,DC=rev,DC=local"
```

## 🔄 Computer Account Management

### Computer Account Modification
```powershell
# Update computer description
Set-ADComputer -Identity "HR-PC-01" -Description "HR Manager Workstation - Windows 10 Enterprise - Updated 2026"

# Change managed by user
Set-ADComputer -Identity "HR-PC-01" -ManagedBy "new.manager"

# Update location
Set-ADComputer -Identity "HR-PC-01" -Location "Main Office - HR Department - Room 101"
```

### Computer Account Transfer
```powershell
# Transfer computer between departments
$computer = Get-ADComputer -Identity "SALES-PC-02"
$targetOU = "OU=HR-Computers,OU=Human Resources,OU=Departments,DC=rev,DC=local"

Move-ADObject -Identity $computer.DistinguishedName -TargetPath $targetOU

# Update computer properties
Set-ADComputer -Identity "SALES-PC-02" -Description "HR Workstation - Windows 10 Enterprise - Transferred from Sales" -ManagedBy "hr.manager"
```

### Computer Account Disable/Enable
```powershell
# Disable computer account
Disable-ADAccount -Identity "OLD-PC-01"

# Enable computer account
Enable-ADAccount -Identity "OLD-PC-01"

# Reset computer account (for domain join issues)
Reset-ComputerMachinePassword -ComputerName "HR-PC-01" -Server "PDC.rev.local"
```

## 📊 Computer Account Reporting

### Computer Inventory Report
```powershell
# Generate computer inventory report
$computers = Get-ADComputer -Filter * -Properties Description, ManagedBy, Location, OperatingSystem, OperatingSystemVersion, LastLogonDate, Created

$report = $computers | Select-Object Name, DNSHostName, Description, ManagedBy, Location, OperatingSystem, OperatingSystemVersion, Enabled, LastLogonDate, Created | Sort-Object Name

$report | Export-Csv -Path "C:\Reports\ComputerInventory.csv" -NoTypeInformation
$report | Out-GridView
```

### Departmental Computer Count
```powershell
# Count computers by department
$computers = Get-ADComputer -Filter * -Properties Description

$hrComputers = $computers | Where-Object {$_.Description -like "*HR*"}
$hkComputers = $computers | Where-Object {$_.Description -like "*Housekeeping*"}
$salesComputers = $computers | Where-Object {$_.Description -like "*Sales*"}
$itComputers = $computers | Where-Object {$_.Description -like "*IT*"}

Write-Host "HR Computers: $($hrComputers.Count)"
Write-Host "Housekeeping Computers: $($hkComputers.Count)"
Write-Host "Sales Computers: $($salesComputers.Count)"
Write-Host "IT Computers: $($itComputers.Count)"
```

### Inactive Computer Accounts
```powershell
# Find inactive computers (90+ days)
$inactiveDate = (Get-Date).AddDays(-90)
$inactiveComputers = Get-ADComputer -Filter {LastLogonDate -lt $inactiveDate -and Enabled -eq $true} -Properties LastLogonDate, Description

$inactiveComputers | Select-Object Name, LastLogonDate, Description | Export-Csv -Path "C:\Reports\InactiveComputers.csv" -NoTypeInformation
```

## 🔒 Security Best Practices

### Computer Account Security
```powershell
# Set computer account password change interval
Set-ADComputer -Identity "HR-PC-01" -PasswordNeverExpires $false

# Configure computer account permissions
$acl = Get-Acl "AD:\CN=HR-PC-01,OU=HR-Computers,OU=Human Resources,OU=Departments,DC=rev,DC=local"
# Add specific permissions as needed
Set-Acl "AD:\CN=HR-PC-01,OU=HR-Computers,OU=Human Resources,OU=Departments,DC=rev,DC=local" $acl
```

### Domain Join Security
```powershell
# Create domain join security group
New-ADGroup -Name "Domain-Join-Admins" -SamAccountName "Domain-Join-Admins" -GroupCategory Security -GroupScope Global -DisplayName "Domain Join Administrators" -Path "OU=Security Groups,OU=Groups,DC=rev,DC=local" -Description "Users authorized to join computers to domain"

# Delegate domain join permissions
$ouPath = "OU=Workstations,DC=rev,DC=local"
$group = Get-ADGroup "Domain-Join-Admins"

dsacls $ouPath /G "$($group.SID):CA;Create Computer Object;computer"
```

### Computer Account Cleanup
```powershell
# Find and disable stale computer accounts
$staleDate = (Get-Date).AddDays(-180)
$staleComputers = Get-ADComputer -Filter {LastLogonDate -lt $staleDate -and Enabled -eq $true} -Properties LastLogonDate, Description

foreach ($computer in $staleComputers) {
    Write-Host "Disabling stale computer: $($computer.Name)"
    Disable-ADAccount -Identity $computer.DistinguishedName
    
    # Move to disabled OU
    Move-ADObject -Identity $computer.DistinguishedName -TargetPath "OU=Disabled Accounts,OU=Special,DC=rev,DC=local"
}
```

## 🖼️ Screenshots

### Computer Creation
![New Computer Creation](../screenshots/02-active-directory/computer-creation.png)

### Computer Properties
![Computer Properties Dialog](../screenshots/02-active-directory/computer-properties.png)

### Computer Management
![Active Directory Computers](../screenshots/02-active-directory/computer-management.png)

### Domain Join
![Domain Join Process](../screenshots/02-active-directory/domain-join.png)

## 📋 Computer Account Summary

| Category | Computers Created | Purpose |
|----------|-------------------|---------|
| Domain Controllers | 1 | Primary domain controller |
| File Servers | 1 | File and storage services |
| Application Servers | 1 | Business applications |
| Web Servers | 2 | Web services and load balancing |
| HR Workstations | 2 | HR department users |
| Housekeeping Workstations | 2 | Housekeeping department users |
| Sales Workstations | 2 | Sales department users |
| IT Workstations | 2 | IT department users |
| Total Computers | 12 | Complete infrastructure |

### Key Features Implemented
- ✅ Standardized naming conventions
- ✅ Proper OU placement for Group Policy
- ✅ Detailed computer descriptions and management
- ✅ Departmental organization
- ✅ Security best practices
- ✅ Comprehensive reporting capabilities
- ✅ Lifecycle management procedures

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
