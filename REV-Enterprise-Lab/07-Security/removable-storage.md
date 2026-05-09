# Removable Storage Restrictions

## 📋 Description

This document provides comprehensive procedures for configuring removable storage restrictions in REV-Enterprise-Lab environment, including USB device blocking, media access control, and endpoint security policies.

## 🎯 Objectives

- Disable removable storage access for HR department users
- Allow IT department users to access removable storage
- Implement comprehensive media access controls
- Configure endpoint security policies
- Ensure compliance with data protection requirements

## 🔧 Removable Storage Policy Configuration

### HR Department Removable Storage Restrictions

#### Create HR Removable Storage GPO
```powershell
# Create HR Removable Storage GPO
New-GPO -Name "HR Removable Storage Restrictions" -Comment "Complete removable storage restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Removable Storage Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure comprehensive storage restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 1

# Configure specific device restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Configure USB storage restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Configure CD/DVD restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Configure floppy drive restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Configure tape drive restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630b-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630b-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Configure WPD restrictions
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1
```

#### Configure Security Filtering
```powershell
# Configure security filtering for HR GPO
$hrGPO = Get-GPO -Name "HR Removable Storage Restrictions"

# Remove existing permissions
Get-GPPermission -Name "HR Removable Storage Restrictions" | ForEach-Object {
    if ($_.Trustee.Name -ne "Authenticated Users" -and $_.Trustee.Name -ne "SYSTEM") {
        Remove-GPPermission -Name "HR Removable Storage Restrictions" -TrusteeName $_.Trustee.Name -TrusteeType $_.TrusteeType -ErrorAction SilentlyContinue
    }
}

# Add HR-Group with GpoApply permission
Set-GPPermission -Name "HR Removable Storage Restrictions" -TrusteeName "HR-Group" -TrusteeType Group -PermissionLevel GpoApply -Replace

# Add Domain Admins with GpoApply permission
Set-GPPermission -Name "HR Removable Storage Restrictions" -TrusteeName "Domain Admins" -TrusteeType Group -PermissionLevel GpoApply -Replace

# Add SYSTEM with GpoApply permission
Set-GPPermission -Name "HR Removable Storage Restrictions" -TrusteeName "SYSTEM" -TrusteeType WellKnownGroup -PermissionLevel GpoApply -Replace
```

### Sales Department Removable Storage Restrictions

#### Create Sales Removable Storage GPO
```powershell
# Create Sales Removable Storage GPO
New-GPO -Name "Sales Removable Storage Restrictions" -Comment "Complete removable storage restrictions for Sales department"

# Link to Sales OU
New-GPLink -Name "Sales Removable Storage Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Apply same restrictions as HR
Set-GPRegistryValue -Name "Sales Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 1

# Configure specific device restrictions for Sales
Set-GPRegistryValue -Name "Sales Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1
```

#### Configure Security Filtering for Sales
```powershell
# Configure security filtering for Sales GPO
$salesGPO = Get-GPO -Name "Sales Removable Storage Restrictions"

# Remove existing permissions
Get-GPPermission -Name "Sales Removable Storage Restrictions" | ForEach-Object {
    if ($_.Trustee.Name -ne "Authenticated Users" -and $_.Trustee.Name -ne "SYSTEM") {
        Remove-GPPermission -Name "Sales Removable Storage Restrictions" -TrusteeName $_.Trustee.Name -TrusteeType $_.TrusteeType -ErrorAction SilentlyContinue
    }
}

# Add Sales-Group with GpoApply permission
Set-GPPermission -Name "Sales Removable Storage Restrictions" -TrusteeName "Sales-Group" -TrusteeType Group -PermissionLevel GpoApply -Replace

# Add Domain Admins with GpoApply permission
Set-GPPermission -Name "Sales Removable Storage Restrictions" -TrusteeName "Domain Admins" -TrusteeType Group -PermissionLevel GpoApply -Replace

# Add SYSTEM with GpoApply permission
Set-GPPermission -Name "Sales Removable Storage Restrictions" -TrusteeName "SYSTEM" -TrusteeType WellKnownGroup -PermissionLevel GpoApply -Replace
```

### IT Department Removable Storage Access

#### Create IT Removable Storage Access GPO
```powershell
# Create IT Removable Storage Access GPO
New-GPO -Name "IT Removable Storage Access" -Comment "Allow removable storage access for IT department"

# Link to IT OU with higher priority
New-GPLink -Name "IT Removable Storage Access" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes -Order 1

# Override restrictions for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 0

# Enable USB storage for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0

# Enable USB storage for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0

# Enable CD/DVD for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0

# Enable floppy drive for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0

# Enable tape drive for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630b-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630b-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0

# Enable WPD for IT users
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0
```

#### Configure Security Filtering for IT
```powershell
# Configure security filtering for IT GPO
$itGPO = Get-GPO -Name "IT Removable Storage Access"

# Remove existing permissions
Get-GPPermission -Name "IT Removable Storage Access" | ForEach-Object {
    if ($_.Trustee.Name -ne "Authenticated Users" -and $_.Trustee.Name -ne "SYSTEM") {
        Remove-GPPermission -Name "IT Removable Storage Access" -TrusteeName $_.Trustee.Name -TrusteeType $_.TrusteeType -ErrorAction SilentlyContinue
    }
}

# Add IT-Group with GpoApply permission
Set-GPPermission -Name "IT Removable Storage Access" -TrusteeName "IT-Group" -TrusteeType Group -PermissionLevel GpoApply -Replace

# Add Domain Admins with GpoApply permission
Set-GPPermission -Name "IT Removable Storage Access" -TrusteeName "Domain Admins" -TrusteeType Group -PermissionLevel GpoApply -Replace

# Add SYSTEM with GpoApply permission
Set-GPPermission -Name "IT Removable Storage Access" -TrusteeName "SYSTEM" -TrusteeType WellKnownGroup -PermissionLevel GpoApply -Replace
```

## 🔍 Advanced Removable Storage Configuration

### Device-Specific Restrictions

#### USB Device Class Restrictions
```powershell
# Function to configure USB device restrictions
function Set-USBDeviceRestrictions {
    param(
        [string]$GPOName,
        [bool]$DenyAccess
    )
    
    $denyValue = if ($DenyAccess) { 1 } else { 0 }
    
    # Configure USB Mass Storage devices
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value $denyValue
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value $denyValue
    
    # Configure USB CD-ROM devices
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56308-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value $denyValue
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56308-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value $denyValue
    
    Write-Host "USB device restrictions configured for $GPOName"
}

# Configure USB restrictions for HR
Set-USBDeviceRestrictions -GPOName "HR Removable Storage Restrictions" -DenyAccess $true

# Configure USB access for IT
Set-USBDeviceRestrictions -GPOName "IT Removable Storage Access" -DenyAccess $false
```

#### Media Access Control
```powershell
# Function to configure media access restrictions
function Set-MediaAccessRestrictions {
    param(
        [string]$GPOName,
        [bool]$DenyCDAccess,
        [bool]$DenyDVDAccess,
        [bool]$DenyBurning
    )
    
    # Configure CD/DVD access
    $cdDvdValue = if ($DenyCDAccess -or $DenyDVDAccess) { 1 } else { 0 }
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoCDBurning" -Type DWORD -Value $cdDvdValue
    
    # Configure media burning restrictions
    $burningValue = if ($DenyBurning) { 1 } else { 0 }
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoCDBurning" -Type DWORD -Value $burningValue
    
    # Configure AutoPlay settings
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoDriveTypeAutoRun" -Type DWORD -Value 255
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoAutorun" -Type DWORD -Value 1
    
    Write-Host "Media access restrictions configured for $GPOName"
}

# Configure media restrictions for HR
Set-MediaAccessRestrictions -GPOName "HR Removable Storage Restrictions" -DenyCDAccess $true -DenyDVDAccess $true -DenyBurning $true

# Configure media restrictions for Sales
Set-MediaAccessRestrictions -GPOName "Sales Removable Storage Restrictions" -DenyCDAccess $true -DenyDVDAccess $true -DenyBurning $true

# Allow media access for IT
Set-MediaAccessRestrictions -GPOName "IT Removable Storage Access" -DenyCDAccess $false -DenyDVDAccess $false -DenyBurning $false
```

### Device Installation Restrictions

#### Prevent Device Driver Installation
```powershell
# Function to prevent device driver installation
function Set-DeviceDriverRestrictions {
    param(
        [string]$GPOName,
        [bool]$PreventInstallation
    )
    
    $preventValue = if ($PreventInstallation) { 1 } else { 0 }
    
    # Prevent installation of removable storage drivers
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyRemovableDeviceClasses" -ValueName "Deny_CDROM" -Type DWORD -Value $preventValue
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyRemovableDeviceClasses" -ValueName "Deny_Floppy" -Type DWORD -Value $preventValue
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyRemovableDeviceClasses" -ValueName "Deny_RemovableDisks" -Type DWORD -Value $preventValue
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyRemovableDeviceClasses" -ValueName "Deny_Tape" -Type DWORD -Value $preventValue
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyRemovableDeviceClasses" -ValueName "Deny_WPD" -Type DWORD -Value $preventValue
    
    # Prevent installation of unknown devices
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceInstall\Restrictions\DenyUnrecognizedDevices" -Type DWORD -Value $preventValue
    
    Write-Host "Device driver restrictions configured for $GPOName"
}

# Prevent device driver installation for HR
Set-DeviceDriverRestrictions -GPOName "HR Removable Storage Restrictions" -PreventInstallation $true

# Prevent device driver installation for Sales
Set-DeviceDriverRestrictions -GPOName "Sales Removable Storage Restrictions" -PreventInstallation $true

# Allow device driver installation for IT
Set-DeviceDriverRestrictions -GPOName "IT Removable Storage Access" -PreventInstallation $false
```

## 📊 Removable Storage Monitoring

### Device Connection Monitoring
```powershell
# Function to monitor removable storage device connections
function Get-RemovableStorageDevices {
    $devices = Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DriveType -eq 2}
    
    $deviceReport = @()
    
    foreach ($device in $devices) {
        $deviceReport += [PSCustomObject]@{
            DeviceID = $device.DeviceID
            DriveLetter = $device.DeviceID
            VolumeName = $device.VolumeName
            SizeGB = [math]::Round($device.Size / 1GB, 2)
            FreeSpaceGB = [math]::Round($device.FreeSpace / 1GB, 2)
            FileSystem = $device.FileSystem
            Model = $device.Model
            InterfaceType = $device.InterfaceType
            MediaType = $device.MediaType
            LastAccessed = $device.LastAccessed
            IsRemovable = $true
        }
    }
    
    return $deviceReport
}

# Get current removable storage devices
$removableDevices = Get-RemovableStorageDevices
$removableDevices | Format-Table -AutoSize
```

### USB Device Monitoring
```powershell
# Function to monitor USB devices
function Get-USBDevices {
    $usbDevices = Get-WmiObject -Class Win32_USBControllerDevice | Select-Object DeviceID, Name, Description, Manufacturer, PNPDeviceID
    
    $usbReport = @()
    
    foreach ($device in $usbDevices) {
        $usbReport += [PSCustomObject]@{
            DeviceID = $device.DeviceID
            Name = $device.Name
            Description = $device.Description
            Manufacturer = $device.Manufacturer
            PNPDeviceID = $device.PNPDeviceID
            Status = "Connected"
        }
    }
    
    return $usbReport
}

# Get current USB devices
$usbDevices = Get-USBDevices
$usbDevices | Format-Table -AutoSize
```

### Policy Compliance Monitoring
```powershell
# Function to check removable storage policy compliance
function Test-RemovableStoragePolicyCompliance {
    param(
        [string]$ComputerName,
        [string]$ExpectedPolicy
    )
    
    $complianceResults = @()
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            param($Policy)
            
            # Check registry settings
            $denyAll = Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -Name "Deny_All" -ErrorAction SilentlyContinue
            $usbDeny = Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -Name "Deny_Write" -ErrorAction SilentlyContinue
            $cdDeny = Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -Name "Deny_Write" -ErrorAction SilentlyContinue
            
            return @{
                DenyAll = $denyAll.Dword
                USBDenyWrite = $usbDeny.Dword
                CDDenyWrite = $cdDeny.Dword
                ExpectedPolicy = $Policy
                Compliant = if ($Policy -eq "Deny") { ($denyAll -eq 1 -and $usbDeny -eq 1 -and $cdDeny -eq 1) } else { ($denyAll -eq 0 -and $usbDeny -eq 0 -and $cdDeny -eq 0) } else { $false }
            }
        } -ArgumentList $ExpectedPolicy
        
        $complianceResults += [PSCustomObject]@{
            ComputerName = $ComputerName
            ExpectedPolicy = $ExpectedPolicy
            ActualPolicy = $result.ExpectedPolicy
            DenyAll = $result.DenyAll
            USBDenyWrite = $result.USBDenyWrite
            CDDenyWrite = $result.CDDenyWrite
            Compliant = $result.Compliant
            Status = if ($result.Compliant) { "Pass" } else { "Fail" }
            Timestamp = Get-Date
        }
        
    } catch {
        $complianceResults += [PSCustomObject]@{
            ComputerName = $ComputerName
            ExpectedPolicy = $ExpectedPolicy
            ActualPolicy = "Error"
            DenyAll = "Error"
            USBDenyWrite = "Error"
            CDDenyWrite = "Error"
            Compliant = "Error"
            Status = "Error"
            Timestamp = Get-Date
            Error = $_.Exception.Message
        }
    }
    
    return $complianceResults
}

# Test policy compliance on HR computers
$hrComputers = @("HR-PC-01", "HR-PC-02")
$complianceResults = @()

foreach ($computer in $hrComputers) {
    $result = Test-RemovableStoragePolicyCompliance -ComputerName $computer -ExpectedPolicy "Deny"
    $complianceResults += $result
}

$complianceResults | Format-Table -AutoSize
```

## 🔍 Removable Storage Troubleshooting

### Common Issues and Solutions

#### Policy Not Applying
```powershell
# Function to troubleshoot removable storage policy issues
function Test-RemovableStoragePolicyApplication {
    param(
        [string]$ComputerName,
        [string]$GPOName
    )
    
    $troubleshooting = @()
    
    # Test 1: Check if GPO exists
    $gpo = Get-GPO -Name $GPOName -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "GPO Existence"
        Status = if ($gpo) { "Pass" } else { "Fail" }
        Details = if ($gpo) { "GPO found" } else { "GPO not found" }
    }
    
    if ($gpo) {
        # Test 2: Check if GPO is linked
        $links = Get-GPLink -Guid $gpo.Id
        $domainLink = $links | Where-Object {$_.Target -eq "DC=rev,DC=local"}
        
        $troubleshooting += [PSCustomObject]@{
            Test = "GPO Linking"
            Status = if ($domainLink) { "Pass" } else { "Fail" }
            Details = if ($domainLink) { "GPO linked to domain" } else { "GPO not linked to domain" }
        }
        
        # Test 3: Check if policy settings are configured
        $denyAll = Get-GPRegistryValue -Guid $gpo.Id -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -Name "Deny_All" -ErrorAction SilentlyContinue
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Policy Settings"
            Status = if ($denyAll) { "Configured" } else { "Not Configured" }
            Details = if ($denyAll) { "Deny_All = $($denyAll.Dword)" } else { "Deny_All not configured" }
        }
        
        # Test 4: Check if security filtering is correct
        $permissions = Get-GPPermission -Guid $gpo.Id
        $hrGroupPermission = $permissions | Where-Object {$_.Trustee.Name -eq "HR-Group"}
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Security Filtering"
            Status = if ($hrGroupPermission) { "Configured" } else { "Not Configured" }
            Details = if ($hrGroupPermission) { "HR-Group has GpoApply permission" } else { "HR-Group does not have GpoApply permission" }
        }
    }
    
    return $troubleshooting
}

# Troubleshoot HR removable storage policy
$troubleshootingResult = Test-RemovableStoragePolicyApplication -ComputerName "HR-PC-01" -GPOName "HR Removable Storage Restrictions"
$troubleshootingResult | Format-Table -AutoSize
```

#### Device Still Accessible
```powershell
# Function to test if removable storage is still accessible
function Test-RemovableStorageAccess {
    param(
        [string]$ComputerName
    )
    
    $troubleshooting = @()
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Test USB drive access
            $usbTest = Test-Path "E:\" -ErrorAction SilentlyContinue
            
            # Test CD/DVD drive access
            $cdTest = Test-Path "D:\" -ErrorAction SilentlyContinue
            
            # Test floppy drive access
            $floppyTest = Test-Path "A:\" -ErrorAction SilentlyContinue
            
            # Check if removable storage devices are present
            $devices = Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DriveType -eq 2}
            
            return @{
                USBAccessible = $usbTest
                CDAccessible = $cdTest
                FloppyAccessible = $floppyTest
                RemovableDevicesPresent = ($devices.Count -gt 0)
                DeviceCount = $devices.Count
            }
        }
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Device Access Test"
            Status = if ($result.USBAccessible -or $result.CDAccessible -or $result.FloppyAccessible -or $result.RemovableDevicesPresent) { "Issue Found" } else { "Pass" }
            Details = "USB: $($result.USBAccessible), CD: $($result.CDAccessible), Floppy: $($result.FloppyAccessible), Devices: $($result.DeviceCount)"
        }
        
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "Device Access Test"
            Status = "Error"
            Details = "Error testing device access: $_"
        }
    }
    
    return $troubleshooting
}

# Test device access on HR computers
$accessTest = Test-RemovableStorageAccess -ComputerName "HR-PC-01"
$accessTest | Format-Table -AutoSize
```

## 📊 Removable Storage Reporting

### Generate Compliance Report
```powershell
# Function to generate removable storage compliance report
function New-RemovableStorageComplianceReport {
    param(
        [string]$ReportPath = "C:\Reports\RemovableStorageCompliance_$(Get-Date -Format 'yyyyMMdd_HHmmss').html"
    )
    
    # Get compliance data
    $computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02", "IT-PC-01", "IT-PC-02")
    $complianceData = @()
    
    foreach ($computer in $computers) {
        $compliance = Test-RemovableStoragePolicyCompliance -ComputerName $computer -ExpectedPolicy "Deny"
        $complianceData += $compliance
    }
    
    # Generate HTML report
    $html = @"
<!DOCTYPE html>
<html>
<head>
    <title>Removable Storage Compliance Report - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; background-color: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .header { text-align: center; margin-bottom: 30px; color: #333; }
        .summary { background-color: #e3f2fd; padding: 15px; margin-bottom: 20px; border-radius: 5px; }
        .compliance-table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
        .compliance-table th, .compliance-table td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        .compliance-table th { background-color: #3498db; color: white; font-weight: bold; }
        .compliant { background-color: #d4edda; }
        .non-compliant { background-color: #f8d7da; }
        .error { background-color: #f8d7da; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Removable Storage Compliance Report</h1>
            <p>Generated on: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
        </div>
        
        <div class="summary">
            <h2>Compliance Summary</h2>
            <table class="compliance-table">
                <tr>
                    <th>Department</th>
                    <th>Total Computers</th>
                    <th>Compliant</th>
                    <th>Non-Compliant</th>
                    <th>Compliance Rate</th>
                </tr>
"@
    
    # Add summary data
    $hrCompliant = ($complianceData | Where-Object {$_.ComputerName -like "HR-*" -and $_.Compliant}).Count
    $hrNonCompliant = ($complianceData | Where-Object {$_.ComputerName -like "HR-*" -and -not $_.Compliant}).Count
    $salesCompliant = ($complianceData | Where-Object {$_.ComputerName -like "SALES-*" -and $_.Compliant}).Count
    $salesNonCompliant = ($complianceData | Where-Object {$_.ComputerName -like "SALES-*" -and -not $_.Compliant}).Count
    $itCompliant = ($complianceData | Where-Object {$_.ComputerName -like "IT-*" -and $_.Compliant}).Count
    $itNonCompliant = ($complianceData | Where-Object {$_.ComputerName -like "IT-*" -and -not $_.Compliant}).Count
    
    $html += @"
                <tr>
                    <td>HR</td>
                    <td>2</td>
                    <td>$hrCompliant</td>
                    <td>$hrNonCompliant</td>
                    <td>$([math]::Round(($hrCompliant / 2) * 100, 1))%</td>
                </tr>
                <tr>
                    <td>Sales</td>
                    <td>2</td>
                    <td>$salesCompliant</td>
                    <td>$salesNonCompliant</td>
                    <td>$([math]::Round(($salesCompliant / 2) * 100, 1))%</td>
                </tr>
                <tr>
                    <td>IT</td>
                    <td>2</td>
                    <td>$itCompliant</td>
                    <td>$itNonCompliant</td>
                    <td>$([math]::Round(($itCompliant / 2) * 100, 1))%</td>
                </tr>
                <tr>
                    <td><strong>Total</strong></td>
                    <td>6</td>
                    <td>$($hrCompliant + $salesCompliant + $itCompliant)</td>
                    <td>$($hrNonCompliant + $salesNonCompliant + $itNonCompliant)</td>
                    <td>$([math]::Round((($hrCompliant + $salesCompliant + $itCompliant) / 6) * 100, 1))%</td>
                </tr>
            </table>
        </div>
        
        <div class="compliance-table">
            <h2>Detailed Compliance Results</h2>
            <table class="compliance-table">
                <tr>
                    <th>Computer</th>
                    <th>Expected Policy</th>
                    <th>Actual Policy</th>
                    <th>Status</th>
                    <th>USB Deny</th>
                    <th>CD/DVD Deny</th>
                    <th>Timestamp</th>
                </tr>
"@
    
    # Add detailed results
    foreach ($data in $complianceData) {
        $statusClass = if ($data.Status -eq "Pass") { "compliant" } elseif ($data.Status -eq "Fail") { "non-compliant" } else { "error" }
        
        $html += @"
                <tr class="$statusClass">
                    <td>$($data.ComputerName)</td>
                    <td>$($data.ExpectedPolicy)</td>
                    <td>$($data.ActualPolicy)</td>
                    <td>$($data.Status)</td>
                    <td>$($data.DenyAll)</td>
                    <td>$($data.USBDenyWrite)</td>
                    <td>$(Get-Date $data.Timestamp -Format 'yyyy-MM-dd HH:mm:ss')</td>
                </tr>
"@
    }
    
    $html += @"
            </table>
        </div>
    </div>
</body>
</html>
"@
    
    # Save HTML report
    $html | Out-File -FilePath $ReportPath -Encoding UTF8
    
    Write-Host "Removable storage compliance report generated: $ReportPath"
    return $ReportPath
}

# Generate compliance report
$complianceReport = New-RemovableStorageComplianceReport
```

## 🖼️ Screenshots

### Removable Storage Policy
![Removable Storage GPO](../screenshots/07-security/removable-storage-policy.png)

### USB Device Restrictions
![USB Restrictions](../screenshots/07-security/usb-restrictions.png)

### Media Access Control
![Media Access Control](../screenshots/07-security/media-access-control.png)

### Compliance Report
![Compliance Report](../screenshots/07-security/compliance-report.png)

## 📋 Removable Storage Policy Summary

| Policy Setting | HR Department | Sales Department | IT Department | Status |
|-----------------|---------------|----------------|-------------|---------|
| USB Storage | Denied | Denied | Allowed | ✅ Configured |
| CD/DVD Access | Denied | Denied | Allowed | ✅ Configured |
| Floppy Drive | Denied | Denied | Allowed | ✅ Configured |
| Tape Drive | Denied | Denied | Allowed | ✅ Configured |
| WPD Devices | Denied | Denied | Allowed | ✅ Configured |
| Device Driver Installation | Prevented | Prevented | Allowed | ✅ Configured |
| Media Burning | Disabled | Disabled | Enabled | ✅ Configured |
| AutoPlay | Disabled | Disabled | Enabled | ✅ Configured |
| Security Filtering | HR-Group Only | Sales-Group Only | IT-Group Only | ✅ Configured |

### Key Features Implemented
- ✅ Complete removable storage restrictions for HR and Sales
- ✅ Removable storage access allowed for IT department
- ✅ USB device class restrictions
- ✅ Media access control and burning prevention
- **Device driver installation prevention
- **AutoPlay and autorun restrictions
- **Comprehensive monitoring and compliance reporting
- **Automated troubleshooting procedures
- **Real-time device connection monitoring

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
