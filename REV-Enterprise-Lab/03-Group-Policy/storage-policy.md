# Storage Policy

## 📋 Description

This document outlines comprehensive storage policies implemented through Group Policy Objects in the REV-Enterprise-Lab environment, including disk quota management, folder redirection, and storage access controls.

## 🎯 Objectives

- Implement disk quota policies for user storage management
- Configure folder redirection for centralized data storage
- Establish storage access controls and limitations
- Prevent unauthorized storage device usage
- Ensure efficient storage utilization across the organization

## 💾 Disk Quota Policies

### User Disk Quota Configuration

#### HR Department Quota Policy
```powershell
# Create HR Storage Quota GPO
New-GPO -Name "HR Storage Quota Policy" -Comment "Disk quota management for HR department users"

# Link to HR OU
New-GPLink -Name "HR Storage Quota Policy" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure disk quota settings
Set-GPRegistryValue -Name "HR Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Enable" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Enforce" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Limit" -Type DWORD -Value 2048  # 2GB in MB
Set-GPRegistryValue -Name "HR Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "WarningLevel" -Type DWORD -Value 1536  # 1.5GB in MB

# Configure quota logging
Set-GPRegistryValue -Name "HR Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "LogEvent" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "LogThreshold" -Type DWORD -Value 1
```

#### Sales Department Quota Policy
```powershell
# Create Sales Storage Quota GPO
New-GPO -Name "Sales Storage Quota Policy" -Comment "Disk quota management for Sales department users"

# Link to Sales OU
New-GPLink -Name "Sales Storage Quota Policy" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure disk quota settings for Sales
Set-GPRegistryValue -Name "Sales Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Enable" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Enforce" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Limit" -Type DWORD -Value 3072  # 3GB in MB
Set-GPRegistryValue -Name "Sales Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "WarningLevel" -Type DWORD -Value 2560  # 2.5GB in MB

# Configure quota logging
Set-GPRegistryValue -Name "Sales Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "LogEvent" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "LogThreshold" -Type DWORD -Value 1
```

#### IT Department Quota Policy
```powershell
# Create IT Storage Quota GPO
New-GPO -Name "IT Storage Quota Policy" -Comment "Disk quota management for IT department users"

# Link to IT OU
New-GPLink -Name "IT Storage Quota Policy" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure disk quota settings for IT (higher quota)
Set-GPRegistryValue -Name "IT Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Enable" -Type DWORD -Value 1
Set-GPRegistryValue -Name "IT Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Enforce" -Type DWORD -Value 1
Set-GPRegistryValue -Name "IT Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "Limit" -Type DWORD -Value 5120  # 5GB in MB
Set-GPRegistryValue -Name "IT Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "WarningLevel" -Type DWORD -Value 4096  # 4GB in MB

# Configure quota logging
Set-GPRegistryValue -Name "IT Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "LogEvent" -Type DWORD -Value 1
Set-GPRegistryValue -Name "IT Storage Quota Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\DiskQuota" -ValueName "LogThreshold" -Type DWORD -Value 1
```

## 📁 Folder Redirection Policies

### User Profile Folder Redirection

#### HR Department Folder Redirection
```powershell
# Create HR Folder Redirection GPO
New-GPO -Name "HR Folder Redirection" -Comment "Folder redirection for HR department users"

# Link to HR OU
New-GPLink -Name "HR Folder Redirection" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Documents folder redirection
Set-GPRegistryValue -Name "HR Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Personal" -Type String -Value "\\FS01\HR$\%USERNAME%\Documents"

# Configure Desktop folder redirection
Set-GPRegistryValue -Name "HR Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Desktop" -Type String -Value "\\FS01\HR$\%USERNAME%\Desktop"

# Configure Favorites folder redirection
Set-GPRegistryValue -Name "HR Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Favorites" -Type String -Value "\\FS01\HR$\%USERNAME%\Favorites"

# Configure Pictures folder redirection
Set-GPRegistryValue -Name "HR Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "My Pictures" -Type String -Value "\\FS01\HR$\%USERNAME%\Pictures"
```

#### Sales Department Folder Redirection
```powershell
# Create Sales Folder Redirection GPO
New-GPO -Name "Sales Folder Redirection" -Comment "Folder redirection for Sales department users"

# Link to Sales OU
New-GPLink -Name "Sales Folder Redirection" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Documents folder redirection
Set-GPRegistryValue -Name "Sales Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Personal" -Type String -Value "\\FS01\Sales$\%USERNAME%\Documents"

# Configure Desktop folder redirection
Set-GPRegistryValue -Name "Sales Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Desktop" -Type String -Value "\\FS01\Sales$\%USERNAME%\Desktop"

# Configure Favorites folder redirection
Set-GPRegistryValue -Name "Sales Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Favorites" -Type String -Value "\\FS01\Sales$\%USERNAME%\Favorites"
```

#### IT Department Folder Redirection
```powershell
# Create IT Folder Redirection GPO
New-GPO -Name "IT Folder Redirection" -Comment "Folder redirection for IT department users"

# Link to IT OU
New-GPLink -Name "IT Folder Redirection" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Documents folder redirection
Set-GPRegistryValue -Name "IT Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Personal" -Type String -Value "\\FS01\IT$\%USERNAME%\Documents"

# Configure Desktop folder redirection
Set-GPRegistryValue -Name "IT Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Desktop" -Type String -Value "\\FS01\IT$\%USERNAME%\Desktop"

# Configure Favorites folder redirection
Set-GPRegistryValue -Name "IT Folder Redirection" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -ValueName "Favorites" -Type String -Value "\\FS01\IT$\%USERNAME%\Favorites"
```

## 🚫 Removable Storage Policies

### Complete Removable Storage Disable for HR

#### HR Removable Storage Restrictions
```powershell
# Create HR Removable Storage Policy GPO
New-GPO -Name "HR Removable Storage Policy" -Comment "Complete removable storage restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Removable Storage Policy" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable all removable storage devices
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 1

# Disable USB storage devices specifically
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Disable CD/DVD drives
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Disable floppy drives
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56311-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1

# Disable tape drives
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630b-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630b-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 1
```

### IT Department Removable Storage Access

#### IT Removable Storage Policy
```powershell
# Create IT Removable Storage Policy GPO
New-GPO -Name "IT Removable Storage Policy" -Comment "Allow removable storage access for IT department"

# Link to IT OU with higher priority
New-GPLink -Name "IT Removable Storage Policy" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes -Order 1

# Override restrictions for IT
Set-GPRegistryValue -Name "IT Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -ValueName "Deny_All" -Type DWORD -Value 0

# Allow USB storage devices for IT
Set-GPRegistryValue -Name "IT Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Write" -Type DWORD -Value 0
Set-GPRegistryValue -Name "IT Removable Storage Policy" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f56307-b6bf-11d0-94f2-00a0c91efb8b}" -ValueName "Deny_Read" -Type DWORD -Value 0

# Configure security filtering to IT-Group only
Get-GPO "IT Removable Storage Policy" | Set-GPPermission -TargetName "IT-Group" -TargetType Group -PermissionLevel GpoApply -Replace
```

## 📊 Storage Management Procedures

### Folder Structure Creation
```powershell
# Create departmental folder structure on file server
$departments = @("HR", "Sales", "IT", "Housekeeping")
$basePath = "\\FS01\Departmental$"

foreach ($dept in $departments) {
    $deptPath = Join-Path $basePath $dept
    New-Item -Path $deptPath -ItemType Directory -Force
    
    # Create user subfolders
    $users = Get-ADUser -Filter "Department -eq '$dept'"
    foreach ($user in $users) {
        $userPath = Join-Path $deptPath $user.SamAccountName
        New-Item -Path $userPath -ItemType Directory -Force
        
        # Create standard subfolders
        New-Item -Path "$userPath\Documents" -ItemType Directory -Force
        New-Item -Path "$userPath\Desktop" -ItemType Directory -Force
        New-Item -Path "$userPath\Favorites" -ItemType Directory -Force
        New-Item -Path "$userPath\Pictures" -ItemType Directory -Force
        
        # Set permissions
        $acl = Get-Acl $userPath
        $accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($user.SamAccountName, "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
        $acl.SetAccessRule($accessRule)
        Set-Acl $userPath $acl
    }
}
```

### Quota Monitoring Script
```powershell
# Monitor disk quota usage
function Get-DiskQuotaReport {
    param(
        [string]$ComputerName
    )
    
    try {
        $quotaEntries = Get-WmiObject -Class Win32_DiskQuota -Computer $ComputerName
        
        foreach ($quota in $quotaEntries) {
            $user = Get-ADUser -Identity $quota.User.Sid.Value -Properties DisplayName
            
            [PSCustomObject]@{
                UserName = $user.DisplayName
                SamAccountName = $user.SamAccountName
                QuotaLimit = [math]::Round($quota.Limit / 1MB, 2)
                CurrentUsage = [math]::Round($quota.DiskSpaceUsed / 1MB, 2)
                UsagePercent = [math]::Round(($quota.DiskSpaceUsed / $quota.Limit) * 100, 2)
                WarningThreshold = [math]::Round($quota.WarningLimit / 1MB, 2)
                Status = if ($quota.DiskSpaceUsed -gt $quota.WarningLimit) { "Warning" } else { "Normal" }
            }
        }
    } catch {
        Write-Host "Error retrieving quota information from $ComputerName`: $_"
    }
}

# Generate quota report for all computers
$computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02", "IT-PC-01", "IT-PC-02")
$quotaReport = @()

foreach ($computer in $computers) {
    $computerQuotas = Get-DiskQuotaReport -ComputerName $computer
    $quotaReport += $computerQuotas
}

$quotaReport | Export-Csv -Path "C:\Reports\DiskQuotaReport.csv" -NoTypeInformation
$quotaReport | Where-Object {$_.Status -eq "Warning"} | Format-Table -AutoSize
```

### Storage Cleanup Automation
```powershell
# Automated storage cleanup for temporary files
function Invoke-StorageCleanup {
    param(
        [string]$ComputerName,
        [int]$DaysOld = 30
    )
    
    $tempPaths = @(
        "C:\Windows\Temp",
        "C:\Users\*\AppData\Local\Temp",
        "C:\Users\*\AppData\Local\Microsoft\Windows\INetCache",
        "C:\Users\*\AppData\Local\Microsoft\Windows\Temporary Internet Files"
    )
    
    foreach ($path in $tempPaths) {
        try {
            $files = Get-ChildItem -Path $path -Recurse -File -ErrorAction SilentlyContinue | Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-$DaysOld)}
            
            foreach ($file in $files) {
                Remove-Item $file.FullName -Force -ErrorAction SilentlyContinue
                Write-Host "Deleted: $($file.FullName)"
            }
        } catch {
            Write-Host "Error cleaning $path`: $_"
        }
    }
}

# Schedule cleanup task
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-Command 'Invoke-StorageCleanup -ComputerName $env:COMPUTERNAME'"
$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 3am
$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopOnIdleEnd

Register-ScheduledTask -TaskName "Storage Cleanup" -Action $action -Trigger $trigger -Settings $settings -RunLevel Highest
```

## 🔍 Storage Policy Validation

### Test Folder Redirection
```powershell
# Function to test folder redirection
function Test-FolderRedirection {
    param(
        [string]$ComputerName,
        [string]$UserName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            param($User)
            
            # Check if folder redirection is applied
            $documentsPath = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -Name "Personal" -ErrorAction SilentlyContinue
            $desktopPath = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -Name "Desktop" -ErrorAction SilentlyContinue
            
            if ($documentsPath.Personal -like "*\\*") {
                Write-Host "Documents folder redirected to: $($documentsPath.Personal)"
            } else {
                Write-Host "Documents folder not redirected"
            }
            
            if ($desktopPath.Desktop -like "*\\*") {
                Write-Host "Desktop folder redirected to: $($desktopPath.Desktop)"
            } else {
                Write-Host "Desktop folder not redirected"
            }
        } -ArgumentList $UserName
        
    } catch {
        Write-Host "Error testing folder redirection on $ComputerName`: $_"
    }
}

# Test folder redirection on HR computers
Test-FolderRedirection -ComputerName "HR-PC-01" -UserName "john.smith"
Test-FolderRedirection -ComputerName "HR-PC-02" -UserName "sarah.johnson"
```

### Test Removable Storage Restrictions
```powershell
# Function to test removable storage restrictions
function Test-RemovableStorageRestrictions {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Check if removable storage is disabled
            $removableStoragePolicy = Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices" -Name "Deny_All" -ErrorAction SilentlyContinue
            
            if ($removableStoragePolicy.Deny_All -eq 1) {
                Write-Host "Removable storage is disabled on $env:COMPUTERNAME"
            } else {
                Write-Host "Removable storage is enabled on $env:COMPUTERNAME"
            }
        }
        
    } catch {
        Write-Host "Error testing removable storage restrictions on $ComputerName`: $_"
    }
}

# Test restrictions on HR computers
Test-RemovableStorageRestrictions -ComputerName "HR-PC-01"
Test-RemovableStorageRestrictions -ComputerName "HR-PC-02"

# Test IT computers (should be enabled)
Test-RemovableStorageRestrictions -ComputerName "IT-PC-01"
Test-RemovableStorageRestrictions -ComputerName "IT-PC-02"
```

## 🖼️ Screenshots

### Disk Quota Configuration
![Disk Quota Settings](../screenshots/03-group-policy/disk-quota.png)

### Folder Redirection Setup
![Folder Redirection](../screenshots/03-group-policy/folder-redirection.png)

### Removable Storage Restrictions
![Removable Storage Policy](../screenshots/03-group-policy/removable-storage.png)

### Storage Report
![Storage Usage Report](../screenshots/03-group-policy/storage-report.png)

## 📋 Storage Policy Summary

| Policy Type | Departments | GPOs Created | Status |
|-------------|--------------|---------------|---------|
| Disk Quotas | HR, Sales, IT | 3 | ✅ Configured |
| Folder Redirection | HR, Sales, IT | 3 | ✅ Configured |
| Removable Storage | HR (Disabled), IT (Enabled) | 2 | ✅ Configured |
| Storage Management | All | 1 | ✅ Configured |
| Total Policies | Multiple | 9 | ✅ Complete |

### Key Features Implemented
- ✅ Department-specific disk quotas (HR: 2GB, Sales: 3GB, IT: 5GB)
- ✅ Folder redirection for Documents, Desktop, Favorites, Pictures
- ✅ Complete removable storage disable for HR users
- ✅ IT users exempt from removable storage restrictions
- ✅ Automated storage cleanup procedures
- ✅ Comprehensive quota monitoring and reporting
- ✅ Storage policy validation procedures

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
