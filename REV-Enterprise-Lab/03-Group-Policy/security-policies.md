# Security Policies

## 📋 Description

This document outlines the comprehensive security policies implemented through Group Policy Objects (GPOs) in the REV-Enterprise-Lab environment, focusing on user restrictions, system hardening, and access control configurations.

## 🎯 Objectives

- Implement security restrictions for HR and Sales departments
- Configure system access controls and limitations
- Establish user environment restrictions
- Protect sensitive data and system resources
- Ensure compliance with enterprise security standards

## 🔐 User Access Restrictions

### HR Department Security Policies

#### Disable Command Prompt for HR Users
```powershell
# Create HR Security GPO
New-GPO -Name "HR Security Restrictions" -Comment "Security policies for HR department users"

# Link GPO to HR OU
New-GPLink -Name "HR Security Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Command Prompt restriction
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableCMD" -Type DWORD -Value 1

# Configure Command Prompt access from Run dialog
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableRun" -Type DWORD -Value 0

# Configure Command Prompt execution policy
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\PowerShell" -ValueName "EnableScripts" -Type DWORD -Value 0
```

#### Disable Control Panel for HR Users
```powershell
# Disable Control Panel access
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoControlPanel" -Type DWORD -Value 1

# Hide Control Panel from Settings
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSetFolders" -Type DWORD -Value 1

# Disable Settings app access
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSettingsPage" -Type DWORD -Value 1
```

#### Remove Properties from This PC Context Menu
```powershell
# Remove Properties from context menu
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoPropertiesMyComputer" -Type DWORD -Value 1

# Disable access to system properties
Set-GPRegistryValue -Name "HR Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" -ValueName "NoPropertiesMyComputer" -Type DWORD -Value 1
```

### Sales Department Security Policies

#### Sales Security Restrictions GPO
```powershell
# Create Sales Security GPO
New-GPO -Name "Sales Security Restrictions" -Comment "Security policies for Sales department users"

# Link GPO to Sales OU
New-GPLink -Name "Sales Security Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Apply same restrictions as HR
Set-GPRegistryValue -Name "Sales Security Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableCMD" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoControlPanel" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Security Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoPropertiesMyComputer" -Type DWORD -Value 1
```

## 🔧 Removable Storage Policies

### Disable Removable Storage for HR Users
```powershell
# Create Removable Storage GPO
New-GPO -Name "HR Removable Storage Restrictions" -Comment "Disable removable storage access for HR users"

# Link GPO to HR OU
New-GPLink -Name "HR Removable Storage Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable USB storage devices
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 1

# Disable CD/DVD drives
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Disable floppy drives
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1
```

### Exclude IT-Group from Removable Storage Restrictions
```powershell
# Create IT Exclusions GPO
New-GPO -Name "IT Removable Storage Access" -Comment "Allow removable storage access for IT users"

# Link GPO to IT OU with higher priority
New-GPLink -Name "IT Removable Storage Access" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes -Order 1

# Override restrictions for IT
Set-GPRegistryValue -Name "IT Removable Storage Access" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 0

# Configure security filtering to IT-Group only
Get-GPO "IT Removable Storage Access" | Set-GPPermission -TargetName "IT-Group" -TargetType Group -PermissionLevel GpoApply -Replace
```

## 🚫 Software Restriction Policies

### Prevent Executable Files from Running
```powershell
# Create Software Restriction GPO
New-GPO -Name "HR Software Restrictions" -Comment "Prevent unauthorized software execution for HR users"

# Link GPO to HR OU
New-GPLink -Name "HR Software Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure software restriction policy
Set-GPRegistryValue -Name "HR Software Restrictions" -Key "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers" -ValueName "DefaultLevel" -Type DWORD -Value 262144

# Disallowed paths for executables
$disallowedPaths = @(
    "C:\Temp\*.exe",
    "C:\Users\%username%\Downloads\*.exe",
    "C:\Users\%username%\Desktop\*.exe",
    "D:\*.exe",
    "E:\*.exe"
)

foreach ($path in $disallowedPaths) {
    Set-GPRegistryValue -Name "HR Software Restrictions" -Key "HKCU\SOFTWARE\Policies\Microsoft\Windows\Safer\CodeIdentifiers\0\Paths\$path" -ValueName "SaferFlags" -Type DWORD -Value 0
}
```

### Prevent Media Installation
```powershell
# Create Media Installation Restriction GPO
New-GPO -Name "HR Media Installation Restrictions" -Comment "Prevent media installation for HR users"

# Link GPO to HR OU
New-GPLink -Name "HR Media Installation Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Windows Media Player
Set-GPRegistryValue -Name "HR Media Installation Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsMediaPlayer" -ValueName "DisallowMediaSharing" -Type DWORD -Value 1

# Prevent installation from removable media
Set-GPRegistryValue -Name "HR Media Installation Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer" -ValueName "DisableMedia" -Type DWORD -Value 1

# Disable AutoPlay
Set-GPRegistryValue -Name "HR Media Installation Restrictions" -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoDriveTypeAutoRun" -Type DWORD -Value 255
```

## 🔒 System Security Policies

### Password Policy Configuration
```powershell
# Create Default Domain Policy modifications
$defaultDomainGPO = Get-GPO -Name "Default Domain Policy"

# Configure password policy
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MaximumPasswordAge" -Type DWORD -Value 60
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordLength" -Type DWORD -Value 6
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordComplexity" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordHistorySize" -Type DWORD -Value 3
```

### Account Lockout Policy
```powershell
# Configure account lockout policy
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "LockoutBadCount" -Type DWORD -Value 5
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "LockoutDuration" -Type DWORD -Value 30
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "ResetLockoutCounter" -Type DWORD -Value 30
```

### Audit Policy Configuration
```powershell
# Create Audit Policy GPO
New-GPO -Name "Enterprise Audit Policy" -Comment "Comprehensive audit policy for security monitoring"

# Link to domain root
New-GPLink -Name "Enterprise Audit Policy" -Target "DC=rev,DC=local" -LinkEnabled Yes

# Configure audit settings
Set-GPRegistryValue -Name "Enterprise Audit Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" -ValueName "ProcessCreationIncludeCmdLine_Enabled" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Enterprise Audit Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" -ValueName "AuditAccountLogon" -Type DWORD -Value 3
Set-GPRegistryValue -Name "Enterprise Audit Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" -ValueName "AuditAccountManagement" -Type DWORD -Value 3
Set-GPRegistryValue -Name "Enterprise Audit Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" -ValueName "AuditObjectAccess" -Type DWORD -Value 2
```

## 🖥️ Desktop Environment Policies

### Desktop Restrictions
```powershell
# Create Desktop Restrictions GPO
New-GPO -Name "HR Desktop Restrictions" -Comment "Desktop environment restrictions for HR users"

# Link to HR OU
New-GPLink -Name "HR Desktop Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Hide desktop icons
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoDesktop" -Type DWORD -Value 0

# Disable right-click on desktop
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoViewContextMenu" -Type DWORD -Value 1

# Hide Network Neighborhood
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoNetHood" -Type DWORD -Value 1
```

### Start Menu Restrictions
```powershell
# Configure Start Menu restrictions
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoStartMenuMorePrograms" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoFind" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoRun" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Desktop Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSMHelp" -Type DWORD -Value 1
```

## 📊 GPO Management Procedures

### GPO Creation Template
```powershell
# Function to create standardized security GPO
function New-SecurityGPO {
    param(
        [string]$GPOName,
        [string]$Description,
        [string]$TargetOU,
        [hashtable]$RegistrySettings
    )
    
    # Create GPO
    $gpo = New-GPO -Name $GPOName -Comment $Description
    
    # Link to OU
    New-GPLink -Name $GPOName -Target $TargetOU -LinkEnabled Yes
    
    # Apply registry settings
    foreach ($setting in $RegistrySettings.GetEnumerator()) {
        Set-GPRegistryValue -Name $GPOName -Key $setting.Key -ValueName $setting.ValueName -Type $setting.Type -Value $setting.Value
    }
    
    Write-Host "Created security GPO: $GPOName"
}

# Example usage
$settings = @{
    Key = "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"
    ValueName = "NoControlPanel"
    Type = "DWORD"
    Value = 1
}

New-SecurityGPO -GPOName "Test Security GPO" -Description "Test security policy" -TargetOU "OU=Test,DC=rev,DC=local" -RegistrySettings $settings
```

### GPO Validation
```powershell
# Validate GPO application
function Test-GPOApplication {
    param(
        [string]$GPOName,
        [string]$TargetComputer
    )
    
    # Get GPO report
    $report = Get-GPOReport -Name $GPOName -ReportType Xml
    
    # Test on target computer
    $result = Invoke-GPUpdate -Computer $TargetComputer -Force -RandomDelayInMinutes 0
    
    if ($result.GpoStatus -eq "Succeeded") {
        Write-Host "GPO $GPOName applied successfully to $TargetComputer"
    } else {
        Write-Host "GPO $GPOName application failed on $TargetComputer"
    }
}

# Test HR policies
Test-GPOApplication -GPOName "HR Security Restrictions" -TargetComputer "HR-PC-01"
```

## 🔍 GPO Reporting and Monitoring

### GPO Inventory Report
```powershell
# Generate GPO inventory report
$gpos = Get-GPO -All

$report = $gpos | Select-Object DisplayName, Id, CreationTime, ModificationTime, GpoStatus, Description | Sort-Object DisplayName

$report | Export-Csv -Path "C:\Reports\GPOInventory.csv" -NoTypeInformation
$report | Out-GridView
```

### GPO Link Report
```powershell
# Report on GPO links
$links = Get-GPLink -All

foreach ($link in $links) {
    $gpo = Get-GPO -Guid $link.GpoId
    Write-Host "GPO: $($gpo.DisplayName)"
    Write-Host "  Target: $($link.Target)"
    Write-Host "  Enabled: $($link.Enabled)"
    Write-Host "  Order: $($link.Order)"
    Write-Host "  Enforced: $($link.Enforced)"
    Write-Host ""
}
```

### GPO Application Status
```powershell
# Check GPO application status
$computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02")

foreach ($computer in $computers) {
    $result = Get-GPResultantSetOfPolicy -Computer $computer -ReportType Xml
    
    Write-Host "=== $computer ==="
    Write-Host "Applied GPOs:"
    
    $appliedGPOs = $result.GPO | Where-Object {$_.LinkOrder -ne $null}
    foreach ($gpo in $appliedGPOs) {
        Write-Host "  - $($gpo.Name)"
    }
    Write-Host ""
}
```

## 🖼️ Screenshots

### GPO Creation
![GPO Creation Wizard](../screenshots/03-group-policy/gpo-creation.png)

### Security Policy Configuration
![Security Policy Settings](../screenshots/03-group-policy/security-settings.png)

### Registry Policy Settings
![Registry Configuration](../screenshots/03-group-policy/registry-settings.png)

### GPO Linking
![GPO Link Management](../screenshots/03-group-policy/gpo-linking.png)

### GPO Results
![GPResult Output](../screenshots/03-group-policy/gpresult.png)

## 📋 Security Policy Summary

| Policy Category | GPOs Created | Target | Status |
|-----------------|---------------|--------|--------|
| User Access Restrictions | 2 | HR, Sales | ✅ Configured |
| Removable Storage | 2 | HR, IT | ✅ Configured |
| Software Restrictions | 2 | HR, Sales | ✅ Configured |
| System Security | 1 | Domain | ✅ Configured |
| Desktop Environment | 1 | HR | ✅ Configured |
| Total GPOs | 8 | Multiple | ✅ Complete |

### Key Features Implemented
- ✅ Command Prompt disabled for HR and Sales
- ✅ Control Panel disabled for HR and Sales
- ✅ Properties menu removed from This PC
- ✅ Removable storage disabled for HR users
- ✅ IT users excluded from storage restrictions
- ✅ Software execution restrictions implemented
- ✅ Media installation prevention configured
- ✅ Password and account lockout policies set
- ✅ Comprehensive audit policy enabled

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
