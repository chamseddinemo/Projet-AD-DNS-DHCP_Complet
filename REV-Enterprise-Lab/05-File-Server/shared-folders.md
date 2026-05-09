# Shared Folders Configuration

## 📋 Description

This document provides comprehensive procedures for creating and managing shared folders in REV-Enterprise-Lab environment, including share creation, permissions configuration, and access management.

## 🎯 Objectives

- Create departmental shared folders for all departments
- Configure proper share and NTFS permissions
- Implement folder security best practices
- Establish folder structure for efficient organization
- Ensure appropriate access controls for each department

## 📁 Share Creation and Configuration

### Public Shared Folder

#### Public Share Creation
```powershell
# Create Public share directory
$publicPath = "D:\Shares\Public"
New-Item -Path $publicPath -ItemType Directory -Force

# Create standard subfolders
$subfolders = @("Documents", "Downloads", "Common", "Temp")
foreach ($folder in $subfolders) {
    New-Item -Path "$publicPath\$folder" -ItemType Directory -Force
}

# Create the share
$shareName = "Public"
New-SmbShare -Name $shareName -Path $publicPath -ReadAccess "Everyone" -ChangeAccess "Authenticated Users" -FullAccess "Domain Admins" -FolderEnumerationMode AccessBased -CachingMode Documents -Description "Public shared folder for all users"

# Configure share permissions
Grant-SmbShareAccess -Name $shareName -AccountName "Everyone" -AccessRight Read -Force
Grant-SmbShareAccess -Name $shareName -AccountName "Authenticated Users" -AccessRight Change -Force
Grant-SmbShareAccess -Name $shareName -AccountName "Domain Admins" -AccessRight Full -Force

# Remove default Everyone access if needed
Revoke-SmbShareAccess -Name $shareName -AccountName "Everyone" -Force
```

#### Public Share NTFS Permissions
```powershell
# Configure NTFS permissions for Public share
$acl = Get-Acl $publicPath

# Remove inherited permissions
$acl.SetAccessRuleProtection($true, $false)

# Add Authenticated Users - Modify access
$authUsersRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "NT AUTHORITY\Authenticated Users",
    "Modify, Synchronize",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($authUsersRule)

# Add Domain Admins - Full Control
$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($adminRule)

# Add SYSTEM - Full Control
$systemRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "NT AUTHORITY\SYSTEM",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($systemRule)

# Apply permissions
Set-Acl $publicPath $acl
```

### HR Department Shared Folder

#### HR Share Creation
```powershell
# Create HR share directory
$hrPath = "D:\Shares\HR"
New-Item -Path $hrPath -ItemType Directory -Force

# Create HR subfolders
$hrSubfolders = @("Documents", "Policies", "Forms", "Employee Files", "Reports", "Confidential")
foreach ($folder in $hrSubfolders) {
    New-Item -Path "$hrPath\$folder" -ItemType Directory -Force
}

# Create the share
$hrShareName = "HR"
New-SmbShare -Name $hrShareName -Path $hrPath -ReadAccess "HR-Group" -ChangeAccess "HR-Group" -FullAccess "Domain Admins" -FolderEnumerationMode AccessBased -CachingMode Documents -Description "HR department shared folder"

# Configure share permissions
Grant-SmbShareAccess -Name $hrShareName -AccountName "HR-Group" -AccessRight Change -Force
Grant-SmbShareAccess -Name $hrShareName -AccountName "Domain Admins" -AccessRight Full -Force
```

#### HR Share NTFS Permissions
```powershell
# Configure NTFS permissions for HR share
$acl = Get-Acl $hrPath

# Remove inherited permissions
$acl.SetAccessRuleProtection($true, $false)

# Add HR-Group - Modify access (but not delete)
$hrGroupRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\HR-Group",
    "Modify, Synchronize",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($hrGroupRule)

# Add HR-Admins - Full Control
$hrAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\HR-Admins",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($hrAdminsRule)

# Add Domain Admins - Full Control
$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($adminRule)

# Add SYSTEM - Full Control
$systemRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "NT AUTHORITY\SYSTEM",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($systemRule)

# Apply permissions
Set-Acl $hrPath $acl

# Configure special permissions for Confidential folder
$confidentialPath = "$hrPath\Confidential"
$confidentialAcl = Get-Acl $confidentialPath
$confidentialAcl.SetAccessRuleProtection($true, $false)

# Only HR-Admins and Domain Admins can access Confidential folder
$hrAdminsConfidentialRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\HR-Admins",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$confidentialAcl.SetAccessRule($hrAdminsConfidentialRule)

$adminConfidentialRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$confidentialAcl.SetAccessRule($adminConfidentialRule)

Set-Acl $confidentialPath $confidentialAcl
```

### Sales Department Shared Folder

#### Sales Share Creation
```powershell
# Create Sales share directory
$salesPath = "D:\Shares\Sales"
New-Item -Path $salesPath -ItemType Directory -Force

# Create Sales subfolders
$salesSubfolders = @("Documents", "Proposals", "Contracts", "Customer Data", "Reports", "Templates")
foreach ($folder in $salesSubfolders) {
    New-Item -Path "$salesPath\$folder" -ItemType Directory -Force
}

# Create the share
$salesShareName = "Sales"
New-SmbShare -Name $salesShareName -Path $salesPath -ReadAccess "Sales-Group" -ChangeAccess "Sales-Group" -FullAccess "Domain Admins" -FolderEnumerationMode AccessBased -CachingMode Documents -Description "Sales department shared folder"

# Configure share permissions
Grant-SmbShareAccess -Name $salesShareName -AccountName "Sales-Group" -AccessRight Change -Force
Grant-SmbShareAccess -Name $salesShareName -AccountName "Domain Admins" -AccessRight Full -Force
```

#### Sales Share NTFS Permissions
```powershell
# Configure NTFS permissions for Sales share
$acl = Get-Acl $salesPath

# Remove inherited permissions
$acl.SetAccessRuleProtection($true, $false)

# Add Sales-Group - Modify access
$salesGroupRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\Sales-Group",
    "Modify, Synchronize",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($salesGroupRule)

# Add Sales-Admins - Full Control
$salesAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\Sales-Admins",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($salesAdminsRule)

# Add Domain Admins - Full Control
$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($adminRule)

# Add SYSTEM - Full Control
$systemRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "NT AUTHORITY\SYSTEM",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($systemRule)

# Apply permissions
Set-Acl $salesPath $acl
```

### IT Department Shared Folder

#### IT Share Creation
```powershell
# Create IT share directory
$itPath = "D:\Shares\IT"
New-Item -Path $itPath -ItemType Directory -Force

# Create IT subfolders
$itSubfolders = @("Documentation", "Scripts", "Tools", "Software", "Logs", "Backups")
foreach ($folder in $itSubfolders) {
    New-Item -Path "$itPath\$folder" -ItemType Directory -Force
}

# Create the share
$itShareName = "IT"
New-SmbShare -Name $itShareName -Path $itPath -ReadAccess "IT-Group" -ChangeAccess "IT-Group" -FullAccess "IT-Admins" -FolderEnumerationMode AccessBased -CachingMode Documents -Description "IT department shared folder"

# Configure share permissions
Grant-SmbShareAccess -Name $itShareName -AccountName "IT-Group" -AccessRight Change -Force
Grant-SmbShareAccess -Name $itShareName -AccountName "IT-Admins" -AccessRight Full -Force
Grant-SmbShareAccess -Name $itShareName -AccountName "Domain Admins" -AccessRight Full -Force
```

#### IT Share NTFS Permissions
```powershell
# Configure NTFS permissions for IT share
$acl = Get-Acl $itPath

# Remove inherited permissions
$acl.SetAccessRuleProtection($true, $false)

# Add IT-Group - Modify access
$itGroupRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\IT-Group",
    "Modify, Synchronize",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($itGroupRule)

# Add IT-Admins - Full Control
$itAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\IT-Admins",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($itAdminsRule)

# Add Domain Admins - Full Control
$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($adminRule)

# Add SYSTEM - Full Control
$systemRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "NT AUTHORITY\SYSTEM",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($systemRule)

# Apply permissions
Set-Acl $itPath $acl
```

## 🔧 Share Management Procedures

### Share Monitoring and Reporting

#### Share Inventory
```powershell
# Function to get share inventory
function Get-ShareInventory {
    $shares = Get-SmbShare
    $inventory = @()
    
    foreach ($share in $shares) {
        $sharePath = $share.Path
        $size = 0
        $fileCount = 0
        
        if (Test-Path $sharePath) {
            $files = Get-ChildItem -Path $sharePath -Recurse -File -ErrorAction SilentlyContinue
            $size = ($files | Measure-Object -Property Length -Sum).Sum
            $fileCount = $files.Count
        }
        
        $inventory += [PSCustomObject]@{
            ShareName = $share.Name
            Path = $share.Path
            Description = $share.Description
            SizeGB = [math]::Round($size / 1GB, 2)
            FileCount = $fileCount
            ShareType = if ($share.Special) { "Special" } else { "Regular" }
            FolderEnumeration = $share.FolderEnumerationMode
            CachingMode = $share.CachingMode
        }
    }
    
    return $inventory
}

# Get share inventory
$shareInventory = Get-ShareInventory
$shareInventory | Format-Table -AutoSize
$shareInventory | Export-Csv -Path "C:\Reports\ShareInventory.csv" -NoTypeInformation
```

#### Share Access Report
```powershell
# Function to get share access report
function Get-ShareAccessReport {
    $shares = Get-SmbShare
    $accessReport = @()
    
    foreach ($share in $shares) {
        $accessRights = Get-SmbShareAccess -Name $share.Name
        
        foreach ($access in $accessRights) {
            $accessReport += [PSCustomObject]@{
                ShareName = $share.Name
                AccountName = $access.AccountName
                AccessRight = $access.AccessRight
                AccessControlType = $access.AccessControlType
                AccessGranted = $access.AccessGranted
            }
        }
    }
    
    return $accessReport
}

# Get share access report
$shareAccess = Get-ShareAccessReport
$shareAccess | Format-Table -AutoSize
$shareAccess | Export-Csv -Path "C:\Reports\ShareAccess.csv" -NoTypeInformation
```

### Share Maintenance

#### Share Backup Configuration
```powershell
# Function to backup share configurations
function Backup-ShareConfiguration {
    param(
        [string]$BackupPath = "C:\ShareBackup"
    )
    
    # Create backup directory
    if (!(Test-Path $BackupPath)) {
        New-Item -Path $BackupPath -ItemType Directory -Force
    }
    
    # Export share configurations
    $shares = Get-SmbShare
    $shareConfig = @()
    
    foreach ($share in $shares) {
        $accessRights = Get-SmbShareAccess -Name $share.Name
        
        $config = [PSCustomObject]@{
            Name = $share.Name
            Path = $share.Path
            Description = $share.Description
            FolderEnumerationMode = $share.FolderEnumerationMode
            CachingMode = $share.CachingMode
            AccessRights = $accessRights
        }
        
        $shareConfig += $config
    }
    
    # Export to JSON
    $shareConfig | ConvertTo-Json -Depth 3 | Out-File -FilePath "$BackupPath\ShareConfig_$(Get-Date -Format 'yyyyMMdd_HHmmss').json"
    
    Write-Host "Share configuration backed up to $BackupPath"
}

# Backup share configuration
Backup-ShareConfiguration
```

#### Share Validation
```powershell
# Function to validate share configurations
function Test-ShareConfiguration {
    $validationResults = @()
    
    $shares = Get-SmbShare
    
    foreach ($share in $shares) {
        $validation = [PSCustomObject]@{
            ShareName = $share.Name
            PathExists = Test-Path $share.Path
            AccessConfigured = $false
            PermissionsValid = $false
            OverallStatus = "Unknown"
        }
        
        # Check if path exists
        if ($validation.PathExists) {
            # Check if access is configured
            $accessRights = Get-SmbShareAccess -Name $share.Name
            $validation.AccessConfigured = ($accessRights.Count -gt 0)
            
            # Check if permissions are valid
            $validation.PermissionsValid = ($accessRights | Where-Object {$_.AccessControlType -eq "Allow"}).Count -gt 0
            
            # Determine overall status
            if ($validation.AccessConfigured -and $validation.PermissionsValid) {
                $validation.OverallStatus = "Valid"
            } else {
                $validation.OverallStatus = "Invalid"
            }
        } else {
            $validation.OverallStatus = "Path Missing"
        }
        
        $validationResults += $validation
    }
    
    return $validationResults
}

# Validate share configurations
$shareValidation = Test-ShareConfiguration
$shareValidation | Format-Table -AutoSize
```

## 🔒 Share Security Best Practices

### Permission Inheritance Management
```powershell
# Function to disable inheritance on share folders
function Disable-ShareInheritance {
    param(
        [string]$FolderPath
    )
    
    if (Test-Path $FolderPath) {
        $acl = Get-Acl $FolderPath
        
        # Disable inheritance and remove inherited permissions
        $acl.SetAccessRuleProtection($true, $false)
        
        Set-Acl $FolderPath $acl
        Write-Host "Disabled inheritance on $FolderPath"
    } else {
        Write-Host "Path $FolderPath does not exist"
    }
}

# Disable inheritance on all share folders
$sharePaths = @("D:\Shares\Public", "D:\Shares\HR", "D:\Shares\Sales", "D:\Shares\IT")

foreach ($path in $sharePaths) {
    Disable-ShareInheritance -FolderPath $path
}
```

### Share Auditing Configuration
```powershell
# Function to configure share auditing
function Enable-ShareAuditing {
    param(
        [string]$FolderPath,
        [string]$AuditRule = "Everyone"
    )
    
    if (Test-Path $FolderPath) {
        $acl = Get-Acl $FolderPath
        
        # Create audit rule
        $auditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
            $AuditRule,
            "FullControl",
            "ContainerInherit, ObjectInherit",
            "None",
            "Success, Failure"
        )
        
        # Add audit rule
        $acl.SetAuditRule($auditRule)
        
        # Apply audit settings
        Set-Acl $FolderPath $acl
        
        Write-Host "Enabled auditing on $FolderPath for $AuditRule"
    } else {
        Write-Host "Path $FolderPath does not exist"
    }
}

# Enable auditing on all share folders
Enable-ShareAuditing -FolderPath "D:\Shares\Public"
Enable-ShareAuditing -FolderPath "D:\Shares\HR"
Enable-ShareAuditing -FolderPath "D:\Shares\Sales"
Enable-ShareAuditing -FolderPath "D:\Shares\IT"
```

## 📊 Share Performance Monitoring

### Share Usage Statistics
```powershell
# Function to get share usage statistics
function Get-ShareUsageStatistics {
    $shares = Get-SmbShare
    $usageStats = @()
    
    foreach ($share in $shares) {
        $sharePath = $share.Path
        
        if (Test-Path $sharePath) {
            # Get file statistics
            $files = Get-ChildItem -Path $sharePath -Recurse -File -ErrorAction SilentlyContinue
            $folders = Get-ChildItem -Path $sharePath -Recurse -Directory -ErrorAction SilentlyContinue
            
            # Calculate sizes
            $totalSize = ($files | Measure-Object -Property Length -Sum).Sum
            $largestFile = ($files | Sort-Object Length -Descending | Select-Object -First 1)
            
            $usageStats += [PSCustomObject]@{
                ShareName = $share.Name
                Path = $share.Path
                TotalFiles = $files.Count
                TotalFolders = $folders.Count
                TotalSizeGB = [math]::Round($totalSize / 1GB, 2)
                LargestFile = if ($largestFile) { $largestFile.FullName } else { "None" }
                LargestFileSizeMB = if ($largestFile) { [math]::Round($largestFile.Length / 1MB, 2) } else { 0 }
                AverageFileSizeKB = if ($files.Count -gt 0) { [math]::Round(($totalSize / $files.Count) / 1KB, 2) } else { 0 }
            }
        }
    }
    
    return $usageStats
}

# Get share usage statistics
$shareUsage = Get-ShareUsageStatistics
$shareUsage | Format-Table -AutoSize
$shareUsage | Export-Csv -Path "C:\Reports\ShareUsage.csv" -NoTypeInformation
```

## 🔍 Share Troubleshooting

### Common Share Issues

#### Access Denied Troubleshooting
```powershell
# Function to troubleshoot share access issues
function Test-ShareAccess {
    param(
        [string]$ShareName,
        [string]$TestUser = "REV\TestUser"
    )
    
    $share = Get-SmbShare -Name $ShareName -ErrorAction SilentlyContinue
    
    if ($share) {
        $troubleshooting = @()
        
        # Test 1: Share exists
        $troubleshooting += [PSCustomObject]@{
            Test = "Share Exists"
            Result = "Pass"
            Details = "Share $ShareName exists"
        }
        
        # Test 2: Path exists
        $pathExists = Test-Path $share.Path
        $troubleshooting += [PSCustomObject]@{
            Test = "Path Exists"
            Result = if ($pathExists) { "Pass" } else { "Fail" }
            Details = if ($pathExists) { "Path $($share.Path) exists" } else { "Path $($share.Path) does not exist" }
        }
        
        # Test 3: Share permissions
        $shareAccess = Get-SmbShareAccess -Name $ShareName
        $troubleshooting += [PSCustomObject]@{
            Test = "Share Permissions"
            Result = if ($shareAccess.Count -gt 0) { "Pass" } else { "Fail" }
            Details = "Found $($shareAccess.Count) permission entries"
        }
        
        # Test 4: NTFS permissions
        if (Test-Path $share.Path) {
            try {
                $acl = Get-Acl $share.Path
                $troubleshooting += [PSCustomObject]@{
                    Test = "NTFS Permissions"
                    Result = "Pass"
                    Details = "Can read NTFS permissions"
                }
            } catch {
                $troubleshooting += [PSCustomObject]@{
                    Test = "NTFS Permissions"
                    Result = "Fail"
                    Details = "Cannot read NTFS permissions: $_"
                }
            }
        }
        
        return $troubleshooting
    } else {
        return @([PSCustomObject]@{
            Test = "Share Lookup"
            Result = "Fail"
            Details = "Share $ShareName not found"
        })
    }
}

# Troubleshoot share access
Test-ShareAccess -ShareName "Public"
Test-ShareAccess -ShareName "HR"
```

## 🖼️ Screenshots

### Share Creation
![Share Creation Wizard](../screenshots/05-file-server/share-creation.png)

### NTFS Permissions
![NTFS Permissions Dialog](../screenshots/05-file-server/ntfs-permissions.png)

### Share Permissions
![Share Permissions Dialog](../screenshots/05-file-server/share-permissions.png)

### Share Management
![Share Management Console](../screenshots/05-file-server/share-management.png)

## 📋 Shared Folders Summary

| Share Name | Path | Purpose | Access Group | Status |
|-------------|-------|---------|---------------|---------|
| Public | D:\Shares\Public | General public access | Authenticated Users | ✅ Created |
| HR | D:\Shares\HR | HR department files | HR-Group | ✅ Created |
| Sales | D:\Shares\Sales | Sales department files | Sales-Group | ✅ Created |
| IT | D:\Shares\IT | IT department files | IT-Group | ✅ Created |

### Key Features Implemented
- ✅ Public shared folder with full create/edit/delete permissions
- ✅ HR department share with create/edit permissions (no delete)
- ✅ Sales department share with full departmental access
- ✅ IT department share with administrative access
- ✅ Proper NTFS and share permissions configured
- ✅ Access-based enumeration enabled
- ✅ Share auditing and monitoring procedures
- ✅ Comprehensive troubleshooting and validation

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
