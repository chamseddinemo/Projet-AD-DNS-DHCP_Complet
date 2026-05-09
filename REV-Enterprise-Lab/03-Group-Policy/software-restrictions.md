# Software Restrictions

## 📋 Description

This document provides comprehensive procedures for implementing software restriction policies through Group Policy Objects in REV-Enterprise-Lab environment, focusing on preventing unauthorized software execution and controlling application access.

## 🎯 Objectives

- Implement software restriction policies for HR and Sales departments
- Prevent execution of unauthorized applications
- Control software installation and deployment
- Establish application whitelisting policies
- Protect systems from malicious software execution

## 🚫 Software Restriction Policies

### HR Department Software Restrictions

#### Create HR Software Restriction GPO
```powershell
# Create HR Software Restrictions GPO
New-GPO -Name "HR Software Restrictions" -Comment "Comprehensive software restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Software Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Software Restriction Policies
Set-GPRegistryValue -Name "HR Software Restrictions" -Key "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers" -ValueName "DefaultLevel" -Type DWORD -Value 262144  # Disallowed by default

# Enable Software Restriction Policies
Set-GPRegistryValue -Name "HR Software Restrictions" -Key "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers" -ValueName "PolicyScope" -Type DWORD -Value 0  # All users
```

#### Configure Disallowed Paths
```powershell
# Disallow execution from common user directories
$disallowedPaths = @(
    "C:\Temp\*.exe",
    "C:\Users\%username%\Downloads\*.exe",
    "C:\Users\%username%\Desktop\*.exe",
    "C:\Users\%username%\Documents\*.exe",
    "C:\Users\%username%\AppData\Local\Temp\*.exe",
    "D:\*.exe",
    "E:\*.exe",
    "F:\*.exe"
)

foreach ($path in $disallowedPaths) {
    $regPath = "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers\0\Paths\$([System.Guid]::NewGuid().ToString())"
    Set-GPRegistryValue -Name "HR Software Restrictions" -Key $regPath -ValueName "ItemData" -Type String -Value $path
    Set-GPRegistryValue -Name "HR Software Restrictions" -Key $regPath -ValueName "SaferFlags" -Type DWORD -Value 0  # Disallowed
}
```

#### Configure Allowed Applications
```powershell
# Allow specific business applications
$allowedApps = @(
    "C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\POWERPNT.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\OUTLOOK.EXE",
    "C:\Program Files\Internet Explorer\iexplore.exe",
    "C:\Program Files\Google\Chrome\Application\chrome.exe",
    "C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrobat.exe",
    "C:\Windows\System32\notepad.exe",
    "C:\Windows\System32\calc.exe",
    "C:\Windows\System32\mspaint.exe"
)

foreach ($app in $allowedApps) {
    $regPath = "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers\131072\Paths\$([System.Guid]::NewGuid().ToString())"
    Set-GPRegistryValue -Name "HR Software Restrictions" -Key $regPath -ValueName "ItemData" -Type String -Value $app
    Set-GPRegistryValue -Name "HR Software Restrictions" -Key $regPath -ValueName "SaferFlags" -Type DWORD -Value 131072  # Unrestricted
}
```

#### Configure Hash Rules for Critical Applications
```powershell
# Create hash rules for critical applications (example)
$criticalApps = @(
    "C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE"
)

foreach ($app in $criticalApps) {
    if (Test-Path $app) {
        $hash = Get-FileHash -Path $app -Algorithm SHA256
        $regPath = "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers\131072\Hash\$([System.Guid]::NewGuid().ToString())"
        Set-GPRegistryValue -Name "HR Software Restrictions" -Key $regPath -ValueName "ItemData" -Type String -Value $hash.Hash
        Set-GPRegistryValue -Name "HR Software Restrictions" -Key $regPath -ValueName "SaferFlags" -Type DWORD -Value 131072
    }
}
```

### Sales Department Software Restrictions

#### Create Sales Software Restriction GPO
```powershell
# Create Sales Software Restrictions GPO
New-GPO -Name "Sales Software Restrictions" -Comment "Comprehensive software restrictions for Sales department"

# Link to Sales OU
New-GPLink -Name "Sales Software Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Software Restriction Policies
Set-GPRegistryValue -Name "Sales Software Restrictions" -Key "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers" -ValueName "DefaultLevel" -Type DWORD -Value 262144
Set-GPRegistryValue -Name "Sales Software Restrictions" -Key "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers" -ValueName "PolicyScope" -Type DWORD -Value 0
```

#### Configure Sales-Specific Restrictions
```powershell
# Apply same disallowed paths as HR
$disallowedPaths = @(
    "C:\Temp\*.exe",
    "C:\Users\%username%\Downloads\*.exe",
    "C:\Users\%username%\Desktop\*.exe",
    "C:\Users\%username%\Documents\*.exe",
    "C:\Users\%username%\AppData\Local\Temp\*.exe",
    "D:\*.exe",
    "E:\*.exe",
    "F:\*.exe"
)

foreach ($path in $disallowedPaths) {
    $regPath = "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers\0\Paths\$([System.Guid]::NewGuid().ToString())"
    Set-GPRegistryValue -Name "Sales Software Restrictions" -Key $regPath -ValueName "ItemData" -Type String -Value $path
    Set-GPRegistryValue -Name "Sales Software Restrictions" -Key $regPath -ValueName "SaferFlags" -Type DWORD -Value 0
}

# Allow Sales-specific applications
$salesApps = @(
    "C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\EXCEL.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\POWERPNT.EXE",
    "C:\Program Files\Microsoft Office\root\Office16\OUTLOOK.EXE",
    "C:\Program Files\Internet Explorer\iexplore.exe",
    "C:\Program Files\Google\Chrome\Application\chrome.exe",
    "C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrobat.exe",
    "C:\Program Files\SalesCRM\CRM.exe"  # Sales-specific CRM
)

foreach ($app in $salesApps) {
    $regPath = "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers\131072\Paths\$([System.Guid]::NewGuid().ToString())"
    Set-GPRegistryValue -Name "Sales Software Restrictions" -Key $regPath -ValueName "ItemData" -Type String -Value $app
    Set-GPRegistryValue -Name "Sales Software Restrictions" -Key $regPath -ValueName "SaferFlags" -Type DWORD -Value 131072
}
```

## 📦 Application Control Policies

### Windows Installer Restrictions

#### HR Installer Restrictions
```powershell
# Create HR Installer Restrictions GPO
New-GPO -Name "HR Installer Restrictions" -Comment "Restrict software installation for HR department"

# Link to HR OU
New-GPLink -Name "HR Installer Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Windows Installer
Set-GPRegistryValue -Name "HR Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableMSI" -Type DWORD -Value 1

# Disable installation from removable media
Set-GPRegistryValue -Name "HR Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableMedia" -Type DWORD -Value 1

# Disable installation from network
Set-GPRegistryValue -Name "HR Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableBrowse" -Type DWORD -Value 1

# Disable installation from Internet
Set-GPRegistryValue -Name "HR Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableHTTP" -Type DWORD -Value 1

# Set elevated installation privileges to disabled
Set-GPRegistryValue -Name "HR Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "AlwaysInstallElevated" -Type DWORD -Value 0
```

#### Sales Installer Restrictions
```powershell
# Create Sales Installer Restrictions GPO
New-GPO -Name "Sales Installer Restrictions" -Comment "Restrict software installation for Sales department"

# Link to Sales OU
New-GPLink -Name "Sales Installer Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Apply same installer restrictions as HR
Set-GPRegistryValue -Name "Sales Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableMSI" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableMedia" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableBrowse" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "DisableHTTP" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Installer Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Installer" -ValueName "AlwaysInstallElevated" -Type DWORD -Value 0
```

### AppLocker Policies

#### Create HR AppLocker Policy
```powershell
# Create HR AppLocker GPO
New-GPO -Name "HR AppLocker Policy" -Comment "AppLocker configuration for HR department"

# Link to HR OU
New-GPLink -Name "HR AppLocker Policy" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Enable AppLocker
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2" -ValueName "AppxEnabled" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2" -ValueName "DllEnabled" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2" -ValueName "ExeEnabled" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2" -ValueName "MsiEnabled" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2" -ValueName "ScriptEnabled" -Type DWORD -Value 1

# Configure enforcement mode
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Enforcement" -ValueName "EnforceDll" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Enforcement" -ValueName "EnforceExe" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR AppLocker Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SrpV2\Enforcement" -ValueName "EnforceMsi" -Type DWORD -Value 1
```

## 🎮 Media and Entertainment Restrictions

### HR Media Restrictions
```powershell
# Create HR Media Restrictions GPO
New-GPO -Name "HR Media Restrictions" -Comment "Restrict media and entertainment applications for HR"

# Link to HR OU
New-GPLink -Name "HR Media Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Windows Media Player
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsMediaPlayer" -ValueName "DisallowMediaSharing" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsMediaPlayer" -ValueName "NoMediaLibrary" -Type DWORD -Value 1

# Disable Windows Media Center
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\MediaCenter" -ValueName "DisallowMediaCenter" -Type DWORD -Value 1

# Disable games
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoGames" -Type DWORD -Value 1

# Disable specific media applications
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "DisallowRun" -Type DWORD -Value 1

# Add disallowed media applications
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "1" -Type String -Value "wmplayer.exe"
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "2" -Type String -Value "ehshell.exe"
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "3" -Type String -Value "solitaire.exe"
Set-GPRegistryValue -Name "HR Media Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "4" -Type String -Value "winmine.exe"
```

## 📊 Software Management Procedures

### Software Inventory Collection
```powershell
# Function to collect software inventory
function Get-SoftwareInventory {
    param(
        [string]$ComputerName
    )
    
    try {
        $software = Get-WmiObject -Class Win32_Product -Computer $ComputerName | 
            Select-Object Name, Version, Vendor, InstallDate, Description
        
        [PSCustomObject]@{
            ComputerName = $ComputerName
            SoftwareCount = $software.Count
            SoftwareList = $software
        }
    } catch {
        Write-Host "Error collecting software inventory from $ComputerName`: $_"
    }
}

# Collect inventory from all computers
$computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02", "IT-PC-01", "IT-PC-02")
$inventoryReport = @()

foreach ($computer in $computers) {
    $inventory = Get-SoftwareInventory -ComputerName $computer
    $inventoryReport += $inventory
}

$inventoryReport | Export-Csv -Path "C:\Reports\SoftwareInventory.csv" -NoTypeInformation
```

### Software Restriction Validation
```powershell
# Function to test software restrictions
function Test-SoftwareRestrictions {
    param(
        [string]$ComputerName,
        [string]$TestExecutable
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            param($Executable)
            
            # Try to run the test executable
            $process = Start-Process -FilePath $Executable -PassThru -WindowStyle Hidden
            
            if ($process) {
                Stop-Process -Id $process.Id -Force
                return "Executable allowed - RESTRICTION FAILED"
            } else {
                return "Executable blocked - RESTRICTION SUCCESS"
            }
        } -ArgumentList $TestExecutable
        
        Write-Host "$ComputerName`: $result"
        
    } catch {
        Write-Host "$ComputerName`: Cannot test executable - ACCESS DENIED (Expected)"
    }
}

# Test restrictions on HR computers
Test-SoftwareRestrictions -ComputerName "HR-PC-01" -TestExecutable "C:\Temp\test.exe"
Test-SoftwareRestrictions -ComputerName "HR-PC-02" -TestExecutable "C:\Temp\test.exe"
```

### Software Compliance Monitoring
```powershell
# Function to monitor software compliance
function Get-SoftwareComplianceReport {
    $computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02")
    $complianceReport = @()
    
    foreach ($computer in $computers) {
        $compliance = [PSCustomObject]@{
            ComputerName = $computer
            Department = "Unknown"
            UnauthorizedSoftware = @()
            ComplianceStatus = "Unknown"
        }
        
        # Determine department
        if ($computer -like "HR*") { $compliance.Department = "HR" }
        elseif ($computer -like "SALES*") { $compliance.Department = "Sales" }
        elseif ($computer -like "IT*") { $compliance.Department = "IT" }
        
        # Check for unauthorized software
        try {
            $software = Get-WmiObject -Class Win32_Product -Computer $computer
            
            $unauthorized = $software | Where-Object {
                $_.Name -like "*Game*" -or 
                $_.Name -like "*Media*" -or 
                $_.Name -like "*Torrent*" -or
                $_.Name -like "*Crack*"
            }
            
            $compliance.UnauthorizedSoftware = $unauthorized.Name
            
            if ($unauthorized.Count -gt 0) {
                $compliance.ComplianceStatus = "Non-Compliant"
            } else {
                $compliance.ComplianceStatus = "Compliant"
            }
            
        } catch {
            $compliance.ComplianceStatus = "Error"
        }
        
        $complianceReport += $compliance
    }
    
    return $complianceReport
}

# Generate compliance report
$complianceReport = Get-SoftwareComplianceReport
$complianceReport | Format-Table -AutoSize
$complianceReport | Export-Csv -Path "C:\Reports\SoftwareCompliance.csv" -NoTypeInformation
```

## 🔍 Software Restriction Troubleshooting

### Common Issues and Solutions

#### Application Won't Start
```powershell
# Function to check if application is blocked
function Test-ApplicationBlock {
    param(
        [string]$ComputerName,
        [string]$ApplicationPath
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            param($AppPath)
            
            # Check application event log for block events
            $blockedEvents = Get-WinEvent -LogName "Microsoft-Windows-AppLocker/EXE and DLL" -MaxEvents 10 | 
                Where-Object {$_.Message -like "*blocked*"}
            
            if ($blockedEvents.Count -gt 0) {
                return "Application is blocked by AppLocker"
            } else {
                return "Application not blocked by AppLocker"
            }
        } -ArgumentList $ApplicationPath
        
        return $result
        
    } catch {
        return "Error checking application block status: $_"
    }
}
```

#### Software Installation Issues
```powershell
# Function to check installer restrictions
function Test-InstallerRestrictions {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Check Windows Installer restrictions
            $msiPolicy = Get-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\Installer" -ErrorAction SilentlyContinue
            
            if ($msiPolicy.DisableMSI -eq 1) {
                return "Windows Installer is disabled"
            } else {
                return "Windows Installer is enabled"
            }
        }
        
        return $result
        
    } catch {
        return "Error checking installer restrictions: $_"
    }
}
```

## 🖼️ Screenshots

### Software Restriction Configuration
![Software Restriction Policy](../screenshots/03-group-policy/software-restriction.png)

### AppLocker Rules
![AppLocker Configuration](../screenshots/03-group-policy/applocker-rules.png)

### Installer Restrictions
![Windows Installer Policy](../screenshots/03-group-policy/installer-restrictions.png)

### Compliance Report
![Software Compliance](../screenshots/03-group-policy/software-compliance.png)

## 📋 Software Restriction Summary

| Restriction Type | Departments | GPOs Created | Status |
|------------------|--------------|---------------|---------|
| Software Execution | HR, Sales | 2 | ✅ Configured |
| Application Control | HR | 1 | ✅ Configured |
| Installer Restrictions | HR, Sales | 2 | ✅ Configured |
| Media Restrictions | HR | 1 | ✅ Configured |
| AppLocker Policies | HR | 1 | ✅ Configured |
| Total Policies | Multiple | 7 | ✅ Complete |

### Key Features Implemented
- ✅ Disallowed execution from user directories (Temp, Downloads, Desktop)
- ✅ Whitelisted business applications for productivity
- ✅ Hash rules for critical application verification
- ✅ Windows Installer restrictions for unauthorized installations
- ✅ Media and entertainment application blocking
- ✅ AppLocker configuration for advanced control
- ✅ Comprehensive software inventory and compliance monitoring
- ✅ Troubleshooting procedures for common issues

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
