# Mapped Drives Configuration

## 📋 Description

This document provides comprehensive procedures for configuring mapped drives through Group Policy in REV-Enterprise-Lab environment, focusing on automated drive mapping for departmental users.

## 🎯 Objectives

- Configure automated drive mapping for HR department users
- Implement persistent drive mappings for user convenience
- Establish drive letter consistency across departments
- Configure drive mapping through Group Policy preferences
- Ensure proper access permissions for mapped drives

## 📁 Drive Mapping Configuration

### HR Department Drive Mapping

#### HR Drive H: Mapping
```powershell
# Create HR Drive Mapping GPO
New-GPO -Name "HR Drive Mapping" -Comment "Map H: drive to HR share for HR department"

# Link GPO to HR OU
New-GPLink -Name "HR Drive Mapping" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure drive mapping using Group Policy Preferences
$gpo = Get-GPO -Name "HR Drive Mapping"
$gpoPath = "\\rev.local\SysVol\rev.local\Policies\{$($gpo.Id)}\User\Preferences\Drives"

# Create drives.xml configuration
$drivesXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Drives clsid="{8FDDCC64-2987-4e6c-A933-5B7B8F4F7A69}">
    <Drive clsid="{935D1B74-C0F1-4424-9E4C-3604832E97F5}" name="H:" status="H:" image="0" changed="2026-05-09 17:45:00" uid="{$(New-Guid)}">
        <Properties action="U" thisDrive="H:" allDrives="FALSE" persistent="TRUE" label="HR Department" useLetter="TRUE" letter="H"></Properties>
    </Drive>
</Drives>
"@

# Create the XML file
New-Item -Path $gpoPath -ItemType Directory -Force
$drivesXml | Out-File -FilePath "$gpoPath\Drives.xml" -Encoding UTF8

# Configure registry settings for drive mapping
Set-GPRegistryValue -Name "HR Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##HR#rev.local\HR" -ValueName "_LabelFromReg" -Type String -Value "HR Department"
Set-GPRegistryValue -Name "HR Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##HR#rev.local\HR" -ValueName "_Label" -Type String -Value "HR Department"
```

#### PowerShell Drive Mapping Script
```powershell
# Create PowerShell script for HR drive mapping
$scriptContent = @"
# HR Drive Mapping Script
# Map H: drive to HR share

try {
    # Remove existing mapping if it exists
    if (Test-Path "H:") {
        Write-Host "Removing existing H: drive mapping..."
        net use H: /delete
    }
    
    # Map H: drive to HR share
    Write-Host "Mapping H: drive to \\FS01\HR..."
    net use H: "\\FS01\HR" /persistent:yes
    
    # Verify mapping
    if (Test-Path "H:") {
        Write-Host "H: drive mapped successfully"
        
        # Set drive label
        $shell = New-Object -ComObject Shell.Application
        $shell.NameSpace("H:\").Self.Name = "HR Department"
        
        Write-Host "Drive label set to 'HR Department'"
    } else {
        Write-Host "Failed to map H: drive"
    }
    
} catch {
    Write-Host "Error mapping drive: `$_"
}
"@

# Create script file
$scriptPath = "\\rev.local\SysVol\rev.local\Scripts\MapHRDrive.ps1"
New-Item -Path (Split-Path $scriptPath) -ItemType Directory -Force
$scriptContent | Out-File -FilePath $scriptPath -Encoding UTF8

# Configure GPO to run script
Set-GPRegistryValue -Name "HR Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce" -ValueName "MapHRDrive" -Type String -Value "powershell.exe -ExecutionPolicy Bypass -File `"$scriptPath`""
```

### Sales Department Drive Mapping

#### Sales Drive S: Mapping
```powershell
# Create Sales Drive Mapping GPO
New-GPO -Name "Sales Drive Mapping" -Comment "Map S: drive to Sales share for Sales department"

# Link GPO to Sales OU
New-GPLink -Name "Sales Drive Mapping" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure drive mapping using Group Policy Preferences
$salesGpo = Get-GPO -Name "Sales Drive Mapping"
$salesGpoPath = "\\rev.local\SysVol\rev.local\Policies\{$($salesGpo.Id)}\User\Preferences\Drives"

# Create drives.xml configuration for Sales
$salesDrivesXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Drives clsid="{8FDDCC64-2987-4e6c-A933-5B7B8F4F7A69}">
    <Drive clsid="{935D1B74-C0F1-4424-9E4C-3604832E97F5}" name="S:" status="S:" image="0" changed="2026-05-09 17:45:00" uid="{$(New-Guid)}">
        <Properties action="U" thisDrive="S:" allDrives="FALSE" persistent="TRUE" label="Sales Department" useLetter="TRUE" letter="S"></Properties>
    </Drive>
</Drives>
"@

# Create the XML file
New-Item -Path $salesGpoPath -ItemType Directory -Force
$salesDrivesXml | Out-File -FilePath "$salesGpoPath\Drives.xml" -Encoding UTF8

# Configure registry settings for Sales drive mapping
Set-GPRegistryValue -Name "Sales Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##Sales#rev.local\Sales" -ValueName "_LabelFromReg" -Type String -Value "Sales Department"
Set-GPRegistryValue -Name "Sales Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##Sales#rev.local\Sales" -ValueName "_Label" -Type String -Value "Sales Department"
```

### IT Department Drive Mapping

#### IT Drive I: Mapping
```powershell
# Create IT Drive Mapping GPO
New-GPO -Name "IT Drive Mapping" -Comment "Map I: drive to IT share for IT department"

# Link GPO to IT OU
New-GPLink -Name "IT Drive Mapping" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure drive mapping using Group Policy Preferences
$itGpo = Get-GPO -Name "IT Drive Mapping"
$itGpoPath = "\\rev.local\SysVol\rev.local\Policies\{$($itGpo.Id)}\User\Preferences\Drives"

# Create drives.xml configuration for IT
$itDrivesXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Drives clsid="{8FDDCC64-2987-4e6c-A933-5B7B8F4F7A69}">
    <Drive clsid="{935D1B74-C0F1-4424-9E4C-3604832E97F5}" name="I:" status="I:" image="0" changed="2026-05-09 17:45:00" uid="{$(New-Guid)}">
        <Properties action="U" thisDrive="I:" allDrives="FALSE" persistent="TRUE" label="IT Department" useLetter="TRUE" letter="I"></Properties>
    </Drive>
</Drives>
"@

# Create the XML file
New-Item -Path $itGpoPath -ItemType Directory -Force
$itDrivesXml | Out-File -FilePath "$itGpoPath\Drives.xml" -Encoding UTF8

# Configure registry settings for IT drive mapping
Set-GPRegistryValue -Name "IT Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##IT#rev.local\IT" -ValueName "_LabelFromReg" -Type String -Value "IT Department"
Set-GPRegistryValue -Name "IT Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##IT#rev.local\IT" -ValueName "_Label" -Type String -Value "IT Department"
```

### Public Drive P: Mapping

#### Public Drive for All Users
```powershell
# Create Public Drive Mapping GPO
New-GPO -Name "Public Drive Mapping" -Comment "Map P: drive to Public share for all users"

# Link GPO to domain root
New-GPLink -Name "Public Drive Mapping" -Target "DC=rev,DC=local" -LinkEnabled Yes

# Configure drive mapping using Group Policy Preferences
$publicGpo = Get-GPO -Name "Public Drive Mapping"
$publicGpoPath = "\\rev.local\SysVol\rev.local\Policies\{$($publicGpo.Id)}\User\Preferences\Drives"

# Create drives.xml configuration for Public
$publicDrivesXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Drives clsid="{8FDDCC64-2987-4e6c-A933-5B7B8F4F7A69}">
    <Drive clsid="{935D1B74-C0F1-4424-9E4C-3604832E97F5}" name="P:" status="P:" image="0" changed="2026-05-09 17:45:00" uid="{$(New-Guid)}">
        <Properties action="U" thisDrive="P:" allDrives="FALSE" persistent="TRUE" label="Public" useLetter="TRUE" letter="P"></Properties>
    </Drive>
</Drives>
"@

# Create the XML file
New-Item -Path $publicGpoPath -ItemType Directory -Force
$publicDrivesXml | Out-File -FilePath "$publicGpoPath\Drives.xml" -Encoding UTF8

# Configure registry settings for Public drive mapping
Set-GPRegistryValue -Name "Public Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##Public#rev.local\Public" -ValueName "_LabelFromReg" -Type String -Value "Public"
Set-GPRegistryValue -Name "Public Drive Mapping" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\##Public#rev.local\Public" -ValueName "_Label" -Type String -Value "Public"
```

## 🔧 Drive Mapping Management

### Drive Mapping Validation Script
```powershell
# Function to validate drive mappings
function Test-DriveMappings {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            $driveMappings = @()
            
            # Check for mapped drives
            $mappedDrives = Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DriveType -eq 4}
            
            foreach ($drive in $mappedDrives) {
                $driveInfo = [PSCustomObject]@{
                    DriveLetter = $drive.DeviceID
                    VolumeName = $drive.VolumeName
                    ProviderName = $drive.ProviderName
                    SizeGB = [math]::Round($drive.Size / 1GB, 2)
                    FreeSpaceGB = [math]::Round($drive.FreeSpace / 1GB, 2)
                    Status = "Mapped"
                }
                
                $driveMappings += $driveInfo
            }
            
            # Check for expected drives
            $expectedDrives = @("H:", "S:", "I:", "P:")
            
            foreach ($expectedDrive in $expectedDrives) {
                $mapped = $mappedDrives | Where-Object {$_.DeviceID -eq $expectedDrive}
                
                if (-not $mapped) {
                    $driveMappings += [PSCustomObject]@{
                        DriveLetter = $expectedDrive
                        VolumeName = "Not Mapped"
                        ProviderName = "N/A"
                        SizeGB = 0
                        FreeSpaceGB = 0
                        Status = "Missing"
                    }
                }
            }
            
            return $driveMappings
        }
        
        return $result
        
    } catch {
        Write-Host "Error testing drive mappings on $ComputerName`: $_"
        return @()
    }
}

# Test drive mappings on HR computers
$hrDriveMappings = Test-DriveMappings -ComputerName "HR-PC-01"
$hrDriveMappings | Format-Table -AutoSize
```

### Drive Mapping Troubleshooting Script
```powershell
# Function to troubleshoot drive mapping issues
function Troubleshoot-DriveMapping {
    param(
        [string]$ComputerName,
        [string]$DriveLetter,
        [string]$SharePath
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            param($Drive, $Share)
            
            $troubleshooting = @()
            
            # Test 1: Check if drive is already mapped
            $existingDrive = Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DeviceID -eq $Drive}
            $troubleshooting += [PSCustomObject]@{
                Test = "Drive Existence"
                Status = if ($existingDrive) { "Exists" } else { "Not Mapped" }
                Details = if ($existingDrive) { "Drive $Drive is already mapped" } else { "Drive $Drive is not mapped" }
            }
            
            # Test 2: Check network connectivity to share
            $shareServer = $Share.Split('\')[2]
            $pingResult = Test-Connection -ComputerName $shareServer -Count 2 -Quiet
            $troubleshooting += [PSCustomObject]@{
                Test = "Network Connectivity"
                Status = if ($pingResult) { "Connected" } else { "Not Connected" }
                Details = "Connection to $shareServer`: $(if ($pingResult) { 'Success' } else { 'Failed' })"
            }
            
            # Test 3: Check share accessibility
            try {
                $shareAccessible = Test-Path $Share
                $troubleshooting += [PSCustomObject]@{
                    Test = "Share Accessibility"
                    Status = if ($shareAccessible) { "Accessible" } else { "Not Accessible" }
                    Details = "Share $Share`: $(if ($shareAccessible) { 'Accessible' } else { 'Not accessible' })"
                }
            } catch {
                $troubleshooting += [PSCustomObject]@{
                    Test = "Share Accessibility"
                    Status = "Error"
                    Details = "Error accessing share: $_"
                }
            }
            
            # Test 4: Check user permissions
            if ($shareAccessible) {
                try {
                    $testFile = "$Share\test_$(Get-Date -Format 'yyyyMMdd_HHmmss').txt"
                    "Test" | Out-File -FilePath $testFile -ErrorAction Stop
                    Remove-Item $testFile -ErrorAction Stop
                    $troubleshooting += [PSCustomObject]@{
                        Test = "Write Permissions"
                        Status = "Granted"
                        Details = "User has write permissions on share"
                    }
                } catch {
                    $troubleshooting += [PSCustomObject]@{
                        Test = "Write Permissions"
                        Status = "Denied"
                        Details = "User does not have write permissions: $_"
                    }
                }
            }
            
            return $troubleshooting
        } -ArgumentList $DriveLetter, $SharePath
        
        return $result
        
    } catch {
        Write-Host "Error troubleshooting drive mapping on $ComputerName`: $_"
        return @()
    }
}

# Troubleshoot HR drive mapping
$troubleshootingResult = Troubleshoot-DriveMapping -ComputerName "HR-PC-01" -DriveLetter "H:" -SharePath "\\FS01\HR"
$troubleshootingResult | Format-Table -AutoSize
```

### Drive Mapping Health Monitoring
```powershell
# Function to monitor drive mapping health
function Get-DriveMappingHealth {
    $computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02", "IT-PC-01", "IT-PC-02")
    $healthReport = @()
    
    foreach ($computer in $computers) {
        try {
            $driveMappings = Test-DriveMappings -ComputerName $computer
            
            $health = [PSCustomObject]@{
                ComputerName = $computer
                Department = switch -Wildcard ($computer) {
                    "HR-*" { "HR" }
                    "SALES-*" { "Sales" }
                    "IT-*" { "IT" }
                    default { "Unknown" }
                }
                ExpectedDrives = 0
                MappedDrives = 0
                MissingDrives = @()
                OverallStatus = "Unknown"
            }
            
            # Determine expected drives based on department
            switch ($health.Department) {
                "HR" { $health.ExpectedDrives = 2 }  # H: and P:
                "Sales" { $health.ExpectedDrives = 2 }  # S: and P:
                "IT" { $health.ExpectedDrives = 2 }  # I: and P:
            }
            
            # Count mapped drives
            $health.MappedDrives = ($driveMappings | Where-Object {$_.Status -eq "Mapped"}).Count
            
            # Identify missing drives
            $missingDrives = $driveMappings | Where-Object {$_.Status -eq "Missing"}
            $health.MissingDrives = $missingDrives.DriveLetter
            
            # Determine overall status
            if ($health.MappedDrives -eq $health.ExpectedDrives) {
                $health.OverallStatus = "Healthy"
            } elseif ($health.MappedDrives -gt 0) {
                $health.OverallStatus = "Partial"
            } else {
                $health.OverallStatus = "Failed"
            }
            
            $healthReport += $health
            
        } catch {
            $healthReport += [PSCustomObject]@{
                ComputerName = $computer
                Department = "Unknown"
                ExpectedDrives = 0
                MappedDrives = 0
                MissingDrives = @()
                OverallStatus = "Error"
            }
        }
    }
    
    return $healthReport
}

# Get drive mapping health report
$driveHealth = Get-DriveMappingHealth
$driveHealth | Format-Table -AutoSize
$driveHealth | Export-Csv -Path "C:\Reports\DriveMappingHealth.csv" -NoTypeInformation
```

## 📊 Drive Mapping Reporting

### Drive Mapping Inventory
```powershell
# Function to get drive mapping inventory
function Get-DriveMappingInventory {
    $computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02", "IT-PC-01", "IT-PC-02")
    $inventory = @()
    
    foreach ($computer in $computers) {
        try {
            $driveMappings = Test-DriveMappings -ComputerName $computer
            
            foreach ($mapping in $driveMappings) {
                $inventory += [PSCustomObject]@{
                    ComputerName = $computer
                    DriveLetter = $mapping.DriveLetter
                    VolumeName = $mapping.VolumeName
                    ProviderName = $mapping.ProviderName
                    SizeGB = $mapping.SizeGB
                    FreeSpaceGB = $mapping.FreeSpaceGB
                    Status = $mapping.Status
                    LastChecked = Get-Date
                }
            }
            
        } catch {
            $inventory += [PSCustomObject]@{
                ComputerName = $computer
                DriveLetter = "Error"
                VolumeName = "Error"
                ProviderName = "Error"
                SizeGB = 0
                FreeSpaceGB = 0
                Status = "Error"
                LastChecked = Get-Date
            }
        }
    }
    
    return $inventory
}

# Get drive mapping inventory
$driveInventory = Get-DriveMappingInventory
$driveInventory | Format-Table -AutoSize
$driveInventory | Export-Csv -Path "C:\Reports\DriveMappingInventory.csv" -NoTypeInformation
```

## 🔍 Drive Mapping Security

### Secure Drive Mapping Configuration
```powershell
# Function to configure secure drive mapping
function Set-SecureDriveMapping {
    param(
        [string]$GPOName,
        [string]$DriveLetter,
        [string]$SharePath,
        [string]$SecurityGroup
    )
    
    # Configure security filtering for GPO
    $gpo = Get-GPO -Name $GPOName
    
    # Remove existing permissions
    Get-GPPermission -Name $GPOName | ForEach-Object {
        if ($_.Trustee.Name -ne "Authenticated Users" -and $_.Trustee.Name -ne "SYSTEM") {
            Remove-GPPermission -Name $GPOName -TrusteeName $_.Trustee.Name -TrusteeType $_.TrusteeType -ErrorAction SilentlyContinue
        }
    }
    
    # Add security group with GpoApply permission
    Set-GPPermission -Name $GPOName -TrusteeName $SecurityGroup -TrusteeType Group -PermissionLevel GpoApply -Replace
    
    # Configure item-level targeting
    $gpoPath = "\\rev.local\SysVol\rev.local\Policies\{$($gpo.Id)}\User\Preferences\Drives"
    
    # Create Groups.xml for item-level targeting
    $groupsXml = @"
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-7E7B-4DD8-A282-5EB4B36A863D}">
    <Group clsid="{6D4A79E4-529C-4FA1-9288-1AE819551644}" name="$SecurityGroup" sid="" image="2" changed="2026-05-09 17:45:00" uid="{$(New-Guid)}" userContext="1" removePolicy="0">
        <Properties action="U" name="" sid="" userContext="1" primaryGroup="0" description="" localGroup="0"/>
    </Group>
</Groups>
"@
    
    $groupsXml | Out-File -FilePath "$gpoPath\Groups.xml" -Encoding UTF8
    
    Write-Host "Configured secure drive mapping for $GPOName"
}

# Configure secure drive mappings
Set-SecureDriveMapping -GPOName "HR Drive Mapping" -DriveLetter "H:" -SharePath "\\FS01\HR" -SecurityGroup "HR-Group"
Set-SecureDriveMapping -GPOName "Sales Drive Mapping" -DriveLetter "S:" -SharePath "\\FS01\Sales" -SecurityGroup "Sales-Group"
Set-SecureDriveMapping -GPOName "IT Drive Mapping" -DriveLetter "I:" -SharePath "\\FS01\IT" -SecurityGroup "IT-Group"
```

## 🖼️ Screenshots

### Drive Mapping GPO
![Drive Mapping GPO](../screenshots/05-file-server/drive-mapping-gpo.png)

### Drive Preferences
![Drive Preferences Configuration](../screenshots/05-file-server/drive-preferences.png)

### Mapped Drives Result
![Mapped Drives](../screenshots/05-file-server/mapped-drives.png)

### Drive Mapping Validation
![Drive Mapping Test](../screenshots/05-file-server/drive-mapping-test.png)

## 📋 Mapped Drives Summary

| Department | Drive Letter | Share Path | Purpose | Status |
|------------|---------------|-------------|---------|---------|
| HR | H: | \\FS01\HR | HR department files | ✅ Configured |
| Sales | S: | \\FS01\Sales | Sales department files | ✅ Configured |
| IT | I: | \\FS01\IT | IT department files | ✅ Configured |
| All Users | P: | \\FS01\Public | Public shared files | ✅ Configured |

### Key Features Implemented
- ✅ HR department H: drive mapping to HR share
- ✅ Sales department S: drive mapping to Sales share
- ✅ IT department I: drive mapping to IT share
- ✅ Public P: drive mapping for all users
- ✅ Persistent drive mappings
- ✅ Custom drive labels for easy identification
- ✅ Security filtering for departmental access
- ✅ Comprehensive validation and troubleshooting procedures

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
