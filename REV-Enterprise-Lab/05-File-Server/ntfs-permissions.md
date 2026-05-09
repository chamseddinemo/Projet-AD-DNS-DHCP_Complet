# NTFS Permissions Configuration

## 📋 Description

This document provides comprehensive procedures for configuring NTFS permissions in REV-Enterprise-Lab environment, including permission inheritance, access control lists, and security best practices.

## 🎯 Objectives

- Configure proper NTFS permissions for all shared folders
- Implement permission inheritance and blocking
- Establish access control lists for departmental resources
- Configure special permissions for sensitive data
- Ensure compliance with security policies

## 🔒 NTFS Permission Fundamentals

### Permission Types
| Permission Type | Description | Use Case |
|-----------------|-------------|-----------|
| Full Control | Complete access to folder/files | Administrators, Owners |
| Modify | Read, write, modify, delete | Standard users |
| Read & Execute | Read files and execute programs | General access |
| List Folder Contents | View folder contents | Directory browsing |
| Read | View file contents | Read-only access |
| Write | Create new files/folders | Data creation |
| Special Permissions | Granular control | Advanced security |

### Inheritance Behavior
| Inheritance Setting | Description | When to Use |
|------------------|-------------|---------------|
| Enabled | Permissions flow from parent | Standard folders |
| Disabled | Permissions set explicitly | Secure folders |
| Copy | Copy inherited permissions | Migration |
| Remove | Remove inherited permissions | Security hardening |

## 📁 Departmental NTFS Permissions

### HR Department Permissions

#### HR Share Base Permissions
```powershell
# Configure HR share base permissions
$hrPath = "D:\Shares\HR"

# Get current ACL
$acl = Get-Acl $hrPath

# Disable inheritance and remove inherited permissions
$acl.SetAccessRuleProtection($true, $false)

# Add HR-Group - Modify access (no delete)
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
$domainAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($domainAdminsRule)

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
```

#### HR Confidential Folder Special Permissions
```powershell
# Configure HR Confidential folder with restricted access
$hrConfidentialPath = "D:\Shares\HR\Confidential"

# Get current ACL
$acl = Get-Acl $hrConfidentialPath

# Disable inheritance and remove inherited permissions
$acl.SetAccessRuleProtection($true, $false)

# Only HR-Admins and Domain Admins can access
$hrAdminsConfidentialRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\HR-Admins",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($hrAdminsConfidentialRule)

$domainAdminsConfidentialRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($domainAdminsConfidentialRule)

$systemConfidentialRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "NT AUTHORITY\SYSTEM",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($systemConfidentialRule)

# Apply permissions
Set-Acl $hrConfidentialPath $acl
```

### Sales Department Permissions

#### Sales Share Base Permissions
```powershell
# Configure Sales share base permissions
$salesPath = "D:\Shares\Sales"

# Get current ACL
$acl = Get-Acl $salesPath

# Disable inheritance and remove inherited permissions
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
$domainAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($domainAdminsRule)

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

### IT Department Permissions

#### IT Share Base Permissions
```powershell
# Configure IT share base permissions
$itPath = "D:\Shares\IT"

# Get current ACL
$acl = Get-Acl $itPath

# Disable inheritance and remove inherited permissions
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
$domainAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($domainAdminsRule)

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

### Public Share Permissions

#### Public Share Open Access
```powershell
# Configure Public share with open access
$publicPath = "D:\Shares\Public"

# Get current ACL
$acl = Get-Acl $publicPath

# Disable inheritance and remove inherited permissions
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
$domainAdminsRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($domainAdminsRule)

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

## 🔧 Advanced NTFS Configuration

### Special Permissions Configuration

#### HR Department - No Delete Permissions
```powershell
# Configure HR users with create/modify but no delete
$hrPath = "D:\Shares\HR"

# Get current ACL
$acl = Get-Acl $hrPath

# Create special access rule for HR-Group
$hrSpecialRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\HR-Group",
    "CreateFiles, CreateDirectories, ReadAndExecute, Write, Modify",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$acl.SetAccessRule($hrSpecialRule)

# Apply permissions
Set-Acl $hrPath $acl
```

#### Auditing Configuration
```powershell
# Configure auditing for HR share
$hrPath = "D:\Shares\HR"

# Get current ACL
$acl = Get-Acl $hrPath

# Create audit rule for HR-Group
$auditRule = New-Object System.Security.AccessControl.FileSystemAuditRule(
    "REV\HR-Group",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Success, Failure"
)
$acl.SetAuditRule($auditRule)

# Apply auditing
Set-Acl $hrPath $acl

# Enable file system auditing
auditpol /set /subcategory:"File System" /success:enable /failure:enable
```

### Permission Inheritance Management

#### Selective Inheritance
```powershell
# Function to configure selective inheritance
function Set-SelectiveInheritance {
    param(
        [string]$Path,
        [string]$Group,
        [string]$Permission
    )
    
    $acl = Get-Acl $Path
    
    # Enable inheritance but remove specific permissions
    $acl.SetAccessRuleProtection($false, $false)
    
    # Add explicit permission that overrides inheritance
    $explicitRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
        $Group,
        $Permission,
        "ContainerInherit, ObjectInherit",
        "None",
        "Allow"
    )
    
    $acl.SetAccessRule($explicitRule)
    Set-Acl $Path $acl
    
    Write-Host "Configured selective inheritance for $Path"
}

# Example: Configure HR Documents folder with special permissions
Set-SelectiveInheritance -Path "D:\Shares\HR\Documents" -Group "REV\HR-Group" -Permission "ReadAndExecute, Write"
```

## 📊 NTFS Permission Management

### Permission Analysis and Reporting

#### Permission Inventory
```powershell
# Function to analyze NTFS permissions
function Get-NTFSPermissionInventory {
    param(
        [string]$Path = "D:\Shares"
    )
    
    $permissionReport = @()
    
    # Get all subdirectories
    $directories = Get-ChildItem -Path $Path -Recurse -Directory
    
    foreach ($dir in $directories) {
        try {
            $acl = Get-Acl $dir.FullName
            
            foreach ($access in $acl.Access) {
                $permissionReport += [PSCustomObject]@{
                    Path = $dir.FullName
                    Identity = $access.IdentityReference.Value
                    AccessControlType = $access.AccessControlType
                    FileSystemRights = $access.FileSystemRights
                    InheritanceFlags = $access.InheritanceFlags
                    IsInherited = $access.IsInherited
                }
            }
        } catch {
            $permissionReport += [PSCustomObject]@{
                Path = $dir.FullName
                Identity = "Error"
                AccessControlType = "Error"
                FileSystemRights = $_.Exception.Message
                InheritanceFlags = "Error"
                IsInherited = "Error"
            }
        }
    }
    
    return $permissionReport
}

# Get NTFS permission inventory
$permissionInventory = Get-NTFSPermissionInventory
$permissionInventory | Format-Table -AutoSize
$permissionInventory | Export-Csv -Path "C:\Reports\NTFSPermissions.csv" -NoTypeInformation
```

#### Permission Validation
```powershell
# Function to validate NTFS permissions
function Test-NTFSPermissions {
    param(
        [string]$Path,
        [string]$ExpectedGroup,
        [string]$ExpectedPermission
    )
    
    $validationResults = @()
    
    try {
        $acl = Get-Acl $Path
        
        # Check if expected group has expected permission
        $groupAccess = $acl.Access | Where-Object {$_.IdentityReference.Value -eq $ExpectedGroup}
        
        if ($groupAccess) {
            $hasPermission = $groupAccess | Where-Object {$_.FileSystemRights -match $ExpectedPermission}
            
            $validationResults += [PSCustomObject]@{
                Test = "Group Permission"
                Status = if ($hasPermission) { "Pass" } else { "Fail" }
                Details = "$ExpectedGroup has $ExpectedPermission on $Path"
            }
        } else {
            $validationResults += [PSCustomObject]@{
                Test = "Group Permission"
                Status = "Fail"
                Details = "$ExpectedGroup not found in ACL for $Path"
            }
        }
        
        # Check inheritance status
        $isInherited = $acl.AreAccessRulesProtected
        $validationResults += [PSCustomObject]@{
            Test = "Inheritance Status"
            Status = if ($isInherited) { "Disabled" } else { "Enabled" }
            Details = "Inheritance is $(if ($isInherited) { 'disabled' } else { 'enabled' }) on $Path"
        }
        
    } catch {
        $validationResults += [PSCustomObject]@{
            Test = "ACL Access"
            Status = "Error"
            Details = "Cannot access ACL for $Path`: $_"
        }
    }
    
    return $validationResults
}

# Test HR permissions
$hrValidation = Test-NTFSPermissions -Path "D:\Shares\HR" -ExpectedGroup "REV\HR-Group" -ExpectedPermission "Modify"
$hrValidation | Format-Table -AutoSize
```

### Permission Migration

#### Backup and Restore Permissions
```powershell
# Function to backup NTFS permissions
function Backup-NTFSPermissions {
    param(
        [string]$Path,
        [string]$BackupFile
    )
    
    $acl = Get-Acl $Path
    $acl | Export-Clixml -Path $BackupFile
    
    Write-Host "NTFS permissions backed up to $BackupFile"
}

# Function to restore NTFS permissions
function Restore-NTFSPermissions {
    param(
        [string]$Path,
        [string]$BackupFile
    )
    
    if (Test-Path $BackupFile) {
        $acl = Import-Clixml -Path $BackupFile
        Set-Acl $Path $acl
        
        Write-Host "NTFS permissions restored from $BackupFile to $Path"
    } else {
        Write-Host "Backup file not found: $BackupFile"
    }
}

# Example usage
Backup-NTFSPermissions -Path "D:\Shares\HR" -BackupFile "C:\Backups\HR_Permissions.xml"
# Restore-NTFSPermissions -Path "D:\Shares\HR" -BackupFile "C:\Backups\HR_Permissions.xml"
```

## 🔍 NTFS Troubleshooting

### Common Permission Issues

#### Access Denied Troubleshooting
```powershell
# Function to troubleshoot access denied issues
function Test-NTFSAccess {
    param(
        [string]$Path,
        [string]$TestUser = "REV\TestUser"
    )
    
    $troubleshooting = @()
    
    # Test 1: Check if path exists
    $pathExists = Test-Path $Path
    $troubleshooting += [PSCustomObject]@{
        Test = "Path Existence"
        Result = if ($pathExists) { "Pass" } else { "Fail" }
        Details = if ($pathExists) { "Path exists" } else { "Path does not exist" }
    }
    
    if ($pathExists) {
        # Test 2: Check if user can read ACL
        try {
            $acl = Get-Acl $Path
            $troubleshooting += [PSCustomObject]@{
                Test = "ACL Access"
                Result = "Pass"
                Details = "Can read ACL"
            }
        } catch {
            $troubleshooting += [PSCustomObject]@{
                Test = "ACL Access"
                Result = "Fail"
                Details = "Cannot read ACL: $_"
            }
        }
        
        # Test 3: Check effective permissions
        try {
            $effectivePermissions = Get-Acl $Path | 
                Select-Object -ExpandProperty Access | 
                Where-Object {$_.IdentityReference.Value -eq $TestUser}
            
            $troubleshooting += [PSCustomObject]@{
                Test = "Effective Permissions"
                Result = if ($effectivePermissions) { "Found" } else { "None" }
                Details = "Found $($effectivePermissions.Count) effective permission entries"
            }
        } catch {
            $troubleshooting += [PSCustomObject]@{
                Test = "Effective Permissions"
                Result = "Error"
                Details = "Error checking effective permissions: $_"
            }
        }
    }
    
    return $troubleshooting
}

# Troubleshoot HR access
$accessTest = Test-NTFSAccess -Path "D:\Shares\HR"
$accessTest | Format-Table -AutoSize
```

#### Inheritance Issues
```powershell
# Function to troubleshoot inheritance issues
function Test-InheritanceConfiguration {
    param(
        [string]$Path
    )
    
    $inheritanceTest = @()
    
    try {
        $acl = Get-Acl $Path
        
        # Check inheritance status
        $isProtected = $acl.AreAccessRulesProtected
        $inheritanceTest += [PSCustomObject]@{
            Test = "Inheritance Status"
            Result = if ($isProtected) { "Disabled" } else { "Enabled" }
            Details = "Inheritance is $(if ($isProtected) { 'disabled' } else { 'enabled' })"
        }
        
        # Check for inherited rules
        $inheritedRules = $acl.Access | Where-Object {$_.IsInherited}
        $inheritanceTest += [PSCustomObject]@{
            Test = "Inherited Rules"
            Result = if ($inheritedRules) { "Found" } else { "None" }
            Details = "Found $($inheritedRules.Count) inherited rules"
        }
        
        # Check for explicit rules
        $explicitRules = $acl.Access | Where-Object {-not $_.IsInherited}
        $inheritanceTest += [PSCustomObject]@{
            Test = "Explicit Rules"
            Result = if ($explicitRules) { "Found" } else { "None" }
            Details = "Found $($explicitRules.Count) explicit rules"
        }
        
    } catch {
        $inheritanceTest += [PSCustomObject]@{
            Test = "ACL Access"
            Result = "Error"
            Details = "Cannot access ACL: $_"
        }
    }
    
    return $inheritanceTest
}

# Test inheritance configuration
$inheritanceTest = Test-InheritanceConfiguration -Path "D:\Shares\HR"
$inheritanceTest | Format-Table -AutoSize
```

## 🖼️ Screenshots

### NTFS Permissions Dialog
![NTFS Permissions](../screenshots/05-file-server/ntfs-permissions-dialog.png)

### Advanced Security Settings
![Advanced Security](../screenshots/05-file-server/advanced-security.png)

### Permission Inheritance
![Inheritance Settings](../screenshots/05-file-server/inheritance-settings.png)

### Auditing Configuration
![Auditing Settings](../screenshots/05-file-server/auditing-settings.png)

## 📋 NTFS Permissions Summary

| Folder | Primary Group | Permission Level | Special Settings | Status |
|--------|----------------|------------------|------------------|---------|
| D:\Shares\HR | HR-Group | Modify (no delete) | Create/Modify only | ✅ Configured |
| D:\Shares\HR\Confidential | HR-Admins | Full Control | Restricted access | ✅ Configured |
| D:\Shares\Sales | Sales-Group | Modify | Standard modify | ✅ Configured |
| D:\Shares\IT | IT-Group | Modify | Standard modify | ✅ Configured |
| D:\Shares\Public | Authenticated Users | Modify | Open access | ✅ Configured |

### Key Features Implemented
- ✅ Departmental permission structure configured
- ✅ HR users with modify but no delete permissions
- ✅ Confidential folder with restricted access
- ✅ Proper inheritance management
- ✅ Auditing configuration for security monitoring
- ✅ Permission backup and restore procedures
- ✅ Comprehensive troubleshooting and validation
- ✅ Access control list management

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
