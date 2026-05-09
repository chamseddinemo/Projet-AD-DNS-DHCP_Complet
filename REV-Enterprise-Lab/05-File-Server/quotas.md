# Disk Quotas Configuration

## 📋 Description

This document provides comprehensive procedures for implementing and managing disk quotas in REV-Enterprise-Lab environment, including quota configuration, monitoring, and enforcement policies.

## 🎯 Objectives

- Implement disk quotas for user storage management
- Configure quota limits for different departments
- Establish quota monitoring and alerting
- Implement quota enforcement and warnings
- Ensure fair storage utilization across organization

## 💾 Quota Configuration

### File Server Quota Setup

#### Enable Quota Management
```powershell
# Install File Server Resource Manager (FSRM) if not already installed
Import-Module ServerManager
Add-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools

# Verify FSRM installation
Get-WindowsFeature -Name FS-Resource-Manager

# Restart FSRM service if needed
Restart-Service -Name Srmsvc -Force
```

#### Configure Quota Templates
```powershell
# Create HR quota template (2GB limit)
$hrTemplate = New-FsrmQuotaTemplate -Name "HR User Quota" -Description "2GB quota for HR users" -Size 2GB -Threshold 80 -ThresholdPercent 80 -Notification $true -EmailTo "admin@rev.local" -ReportType "Html"

# Create Sales quota template (3GB limit)
$salesTemplate = New-FsrmQuotaTemplate -Name "Sales User Quota" -Description "3GB quota for Sales users" -Size 3GB -Threshold 75 -ThresholdPercent 75 -Notification $true -EmailTo "admin@rev.local" -ReportType "Html"

# Create IT quota template (5GB limit)
$itTemplate = New-FsrmQuotaTemplate -Name "IT User Quota" -Description "5GB quota for IT users" -Size 5GB -Threshold 70 -ThresholdPercent 70 -Notification $true -EmailTo "admin@rev.local" -ReportType "Html"

# Create Public quota template (1GB limit)
$publicTemplate = New-FsrmQuotaTemplate -Name "Public Quota" -Description "1GB quota for public folder" -Size 1GB -Threshold 90 -ThresholdPercent 90 -Notification $true -EmailTo "admin@rev.local" -ReportType "Html"

# Display created templates
Get-FsrmQuotaTemplate | Format-Table -AutoSize
```

### Departmental Quota Configuration

#### HR Department Quotas
```powershell
# Configure quotas for HR users
$hrUsers = Get-ADUser -Filter "Department -eq 'Human Resources'" -Properties SamAccountName

foreach ($user in $hrUsers) {
    $userPath = "D:\Shares\HR\$($user.SamAccountName)"
    
    if (Test-Path $userPath) {
        # Apply HR quota template to user folder
        Write-Host "Applying HR quota to $($user.SamAccountName)"
        
        # Create quota using template
        New-FsrmQuota -Path $userPath -Template $hrTemplate -Description "HR quota for $($user.DisplayName)"
    }
}

# Verify HR quotas
$hrQuotas = Get-FsrmQuota | Where-Object {$_.Path -like "*\HR\*"}
$hrQuotas | Select-Object Path, Size, Used, PeakUsage | Format-Table -AutoSize
```

#### Sales Department Quotas
```powershell
# Configure quotas for Sales users
$salesUsers = Get-ADUser -Filter "Department -eq 'Sales'" -Properties SamAccountName

foreach ($user in $salesUsers) {
    $userPath = "D:\Shares\Sales\$($user.SamAccountName)"
    
    if (Test-Path $userPath) {
        # Apply Sales quota template to user folder
        Write-Host "Applying Sales quota to $($user.SamAccountName)"
        
        # Create quota using template
        New-FsrmQuota -Path $userPath -Template $salesTemplate -Description "Sales quota for $($user.DisplayName)"
    }
}

# Verify Sales quotas
$salesQuotas = Get-FsrmQuota | Where-Object {$_.Path -like "*\Sales\*"}
$salesQuotas | Select-Object Path, Size, Used, PeakUsage | Format-Table -AutoSize
```

#### IT Department Quotas
```powershell
# Configure quotas for IT users
$itUsers = Get-ADUser -Filter "Department -eq 'IT'" -Properties SamAccountName

foreach ($user in $itUsers) {
    $userPath = "D:\Shares\IT\$($user.SamAccountName)"
    
    if (Test-Path $userPath) {
        # Apply IT quota template to user folder
        Write-Host "Applying IT quota to $($user.SamAccountName)"
        
        # Create quota using template
        New-FsrmQuota -Path $userPath -Template $itTemplate -Description "IT quota for $($user.DisplayName)"
    }
}

# Verify IT quotas
$itQuotas = Get-FsrmQuota | Where-Object {$_.Path -like "*\IT\*"}
$itQuotas | Select-Object Path, Size, Used, PeakUsage | Format-Table -AutoSize
```

### Public Folder Quotas

#### Public Share Quota
```powershell
# Configure quota for Public share
$publicPath = "D:\Shares\Public"

if (Test-Path $publicPath) {
    # Apply Public quota template
    Write-Host "Applying Public quota to $publicPath"
    
    # Create quota using template
    New-FsrmQuota -Path $publicPath -Template $publicTemplate -Description "Public folder quota"
}

# Verify Public quota
$publicQuotas = Get-FsrmQuota | Where-Object {$_.Path -like "*\Public*"}
$publicQuotas | Select-Object Path, Size, Used, PeakUsage | Format-Table -AutoSize
```

## 🔧 Quota Management Procedures

### Quota Monitoring

#### Real-time Quota Monitoring
```powershell
# Function to get quota usage statistics
function Get-QuotaUsageStatistics {
    $quotas = Get-FsrmQuota
    $statistics = @()
    
    foreach ($quota in $quotas) {
        $usagePercent = if ($quota.Size -gt 0) { [math]::Round(($quota.Used / $quota.Size) * 100, 2) } else { 0 }
        
        $stat = [PSCustomObject]@{
            Path = $quota.Path
            UserName = Split-Path $quota.Path -Leaf
            SizeGB = [math]::Round($quota.Size / 1GB, 2)
            UsedGB = [math]::Round($quota.Used / 1GB, 2)
            FreeGB = [math]::Round(($quota.Size - $quota.Used) / 1GB, 2)
            UsagePercent = $usagePercent
            Status = switch ($usagePercent) {
                {$_ -ge 90} { "Critical" }
                {$_ -ge 80} { "Warning" }
                {$_ -ge 70} { "Caution" }
                default { "Normal" }
            }
            PeakUsageGB = [math]::Round($quota.PeakUsage / 1GB, 2)
            Modified = $quota.Modified
        }
        
        $statistics += $stat
    }
    
    return $statistics
}

# Get quota usage statistics
$quotaStats = Get-QuotaUsageStatistics
$quotaStats | Format-Table -AutoSize
$quotaStats | Export-Csv -Path "C:\Reports\QuotaUsage.csv" -NoTypeInformation
```

#### Quota Alert Configuration
```powershell
# Configure quota notifications
function Set-QuotaNotifications {
    $quotas = Get-FsrmQuota
    
    foreach ($quota in $quotas) {
        # Configure email notification
        Set-FsrmQuota -InputObject $quota -Notification $true -EmailTo "admin@rev.local;it@rev.local" -ReportType "Html"
        
        # Configure command notification for critical usage
        if ($quota.Used / $quota.Size -ge 0.9) {
            $command = "powershell.exe -Command 'Write-EventLog -LogName Application -Source QuotaManager -EventId 1001 -EntryType Warning -Message \"Quota critical: $($quota.Path) is at 90% capacity\"'"
            Set-FsrmQuota -InputObject $quota -NotificationCommand $command -NotificationRunLimitInterval 60
        }
        
        Write-Host "Configured notifications for $($quota.Path)"
    }
}

# Set quota notifications
Set-QuotaNotifications
```

### Quota Reporting

#### Generate Quota Report
```powershell
# Function to generate comprehensive quota report
function New-QuotaReport {
    param(
        [string]$ReportPath = "C:\Reports\QuotaReport_$(Get-Date -Format 'yyyyMMdd_HHmmss').html"
    )
    
    $quotas = Get-FsrmQuota
    $totalSize = ($quotas | Measure-Object -Property Size -Sum).Sum
    $totalUsed = ($quotas | Measure-Object -Property Used -Sum).Sum
    $totalFree = $totalSize - $totalUsed
    
    # Generate HTML report
    $html = @"
<!DOCTYPE html>
<html>
<head>
    <title>Disk Quota Report - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2F2; font-weight: bold; }
        .critical { background-color: #ffebee; }
        .warning { background-color: #fff3e0; }
        .normal { background-color: #e8f5e8; }
        .summary { background-color: #e3f2fd; padding: 15px; margin-bottom: 20px; border-radius: 5px; }
    </style>
</head>
<body>
    <h1>Disk Quota Report</h1>
    <p>Generated on: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
    
    <div class="summary">
        <h2>Summary</h2>
        <p><strong>Total Allocated:</strong> $([math]::Round($totalSize / 1GB, 2)) GB</p>
        <p><strong>Total Used:</strong> $([math]::Round($totalUsed / 1GB, 2)) GB</p>
        <p><strong>Total Free:</strong> $([math]::Round($totalFree / 1GB, 2)) GB</p>
        <p><strong>Overall Usage:</strong> $([math]::Round(($totalUsed / $totalSize) * 100, 2))%</p>
    </div>
    
    <h2>Quota Details</h2>
    <table>
        <tr>
            <th>User/Folder</th>
            <th>Size (GB)</th>
            <th>Used (GB)</th>
            <th>Free (GB)</th>
            <th>Usage %</th>
            <th>Status</th>
            <th>Peak Usage (GB)</th>
        </tr>
"@
    
    foreach ($quota in $quotas) {
        $usagePercent = if ($quota.Size -gt 0) { [math]::Round(($quota.Used / $quota.Size) * 100, 2) } else { 0 }
        $statusClass = switch ($usagePercent) {
            {$_ -ge 90} { "critical" }
            {$_ -ge 80} { "warning" }
            default { "normal" }
        }
        
        $html += @"
        <tr class="$statusClass">
            <td>$(Split-Path $quota.Path -Leaf)</td>
            <td>$([math]::Round($quota.Size / 1GB, 2))</td>
            <td>$([math]::Round($quota.Used / 1GB, 2))</td>
            <td>$([math]::Round(($quota.Size - $quota.Used) / 1GB, 2))</td>
            <td>$usagePercent%</td>
            <td>$(switch ($usagePercent) { {$_ -ge 90} { "Critical" } {$_ -ge 80} { "Warning" } default { "Normal" } })</td>
            <td>$([math]::Round($quota.PeakUsage / 1GB, 2))</td>
        </tr>
"@
    }
    
    $html += @"
    </table>
</body>
</html>
"@
    
    # Save HTML report
    $html | Out-File -FilePath $ReportPath -Encoding UTF8
    
    Write-Host "Quota report generated: $ReportPath"
    return $ReportPath
}

# Generate quota report
$quotaReport = New-QuotaReport
```

### Automated Quota Management

#### Quota Cleanup Script
```powershell
# Function to perform quota cleanup operations
function Invoke-QuotaCleanup {
    $quotas = Get-FsrmQuota
    $cleanupActions = @()
    
    foreach ($quota in $quotas) {
        $cleanupAction = [PSCustomObject]@{
            Path = $quota.Path
            UserName = Split-Path $quota.Path -Leaf
            SizeGB = [math]::Round($quota.Size / 1GB, 2)
            UsedGB = [math]::Round($quota.Used / 1GB, 2)
            UsagePercent = [math]::Round(($quota.Used / $quota.Size) * 100, 2)
            Action = "None"
            FilesProcessed = 0
        }
        
        # Check for cleanup opportunities
        if ($cleanupAction.UsagePercent -ge 85) {
            $cleanupAction.Action = "Cleanup Recommended"
            
            # Find large temporary files
            $tempFiles = Get-ChildItem -Path $quota.Path -Recurse -File -ErrorAction SilentlyContinue | 
                Where-Object {$_.Name -match "temp|tmp|~\$"} | 
                Sort-Object Length -Descending | 
                Select-Object -First 10
            
            foreach ($file in $tempFiles) {
                try {
                    Remove-Item $file.FullName -Force -ErrorAction Stop
                    $cleanupAction.FilesProcessed++
                    Write-Host "Removed temp file: $($file.FullName)"
                } catch {
                    Write-Host "Failed to remove $($file.FullName): $_"
                }
            }
        }
        
        $cleanupActions += $cleanupAction
    }
    
    return $cleanupActions
}

# Perform quota cleanup
$cleanupResults = Invoke-QuotaCleanup
$cleanupResults | Format-Table -AutoSize
```

## 📊 Quota Analysis and Reporting

### Departmental Quota Analysis
```powershell
# Function to analyze quotas by department
function Get-DepartmentalQuotaAnalysis {
    $quotas = Get-FsrmQuota
    $departmentAnalysis = @()
    
    # Group quotas by department
    $hrQuotas = $quotas | Where-Object {$_.Path -like "*\HR\*"}
    $salesQuotas = $quotas | Where-Object {$_.Path -like "*\Sales\*"}
    $itQuotas = $quotas | Where-Object {$_.Path -like "*\IT\*"}
    
    # Analyze HR quotas
    $departmentAnalysis += [PSCustomObject]@{
        Department = "HR"
        UserCount = $hrQuotas.Count
        TotalAllocatedGB = [math]::Round(($hrQuotas | Measure-Object -Property Size -Sum).Sum / 1GB, 2)
        TotalUsedGB = [math]::Round(($hrQuotas | Measure-Object -Property Used -Sum).Sum / 1GB, 2)
        AverageUsagePercent = if ($hrQuotas.Count -gt 0) { [math]::Round(($hrQuotas | ForEach-Object {($_.Used / $_.Size) * 100} | Measure-Object -Average).Average, 2) } else { 0 }
        UsersOverThreshold = ($hrQuotas | Where-Object {($_.Used / $_.Size) -ge 0.8}).Count
    }
    
    # Analyze Sales quotas
    $departmentAnalysis += [PSCustomObject]@{
        Department = "Sales"
        UserCount = $salesQuotas.Count
        TotalAllocatedGB = [math]::Round(($salesQuotas | Measure-Object -Property Size -Sum).Sum / 1GB, 2)
        TotalUsedGB = [math]::Round(($salesQuotas | Measure-Object -Property Used -Sum).Sum / 1GB, 2)
        AverageUsagePercent = if ($salesQuotas.Count -gt 0) { [math]::Round(($salesQuotas | ForEach-Object {($_.Used / $_.Size) * 100} | Measure-Object -Average).Average, 2) } else { 0 }
        UsersOverThreshold = ($salesQuotas | Where-Object {($_.Used / $_.Size) -ge 0.75}).Count
    }
    
    # Analyze IT quotas
    $departmentAnalysis += [PSCustomObject]@{
        Department = "IT"
        UserCount = $itQuotas.Count
        TotalAllocatedGB = [math]::Round(($itQuotas | Measure-Object -Property Size -Sum).Sum / 1GB, 2)
        TotalUsedGB = [math]::Round(($itQuotas | Measure-Object -Property Used -Sum).Sum / 1GB, 2)
        AverageUsagePercent = if ($itQuotas.Count -gt 0) { [math]::Round(($itQuotas | ForEach-Object {($_.Used / $_.Size) * 100} | Measure-Object -Average).Average, 2) } else { 0 }
        UsersOverThreshold = ($itQuotas | Where-Object {($_.Used / $_.Size) -ge 0.7}).Count
    }
    
    return $departmentAnalysis
}

# Get departmental quota analysis
$deptAnalysis = Get-DepartmentalQuotaAnalysis
$deptAnalysis | Format-Table -AutoSize
```

### Quota Trend Analysis
```powershell
# Function to analyze quota usage trends
function Get-QuotaTrendAnalysis {
    $quotas = Get-FsrmQuota
    $trendAnalysis = @()
    
    foreach ($quota in $quotas) {
        # Get historical usage data (if available)
        $usageHistory = Get-FsrmQuota -Path $quota.Path
        
        $trend = [PSCustomObject]@{
            UserName = Split-Path $quota.Path -Leaf
            CurrentUsageGB = [math]::Round($quota.Used / 1GB, 2)
            PeakUsageGB = [math]::Round($quota.PeakUsage / 1GB, 2)
            QuotaLimitGB = [math]::Round($quota.Size / 1GB, 2)
            UsagePercent = [math]::Round(($quota.Used / $quota.Size) * 100, 2)
            Trend = "Stable"  # Would need historical data for real trend analysis
            LastModified = $quota.Modified
            Status = switch ($quota.Used / $quota.Size) {
                {$_ -ge 0.9} { "Critical" }
                {$_ -ge 0.8} { "Warning" }
                {$_ -ge 0.7} { "Caution" }
                default { "Normal" }
            }
        }
        
        $trendAnalysis += $trend
    }
    
    return $trendAnalysis
}

# Get quota trend analysis
$trendAnalysis = Get-QuotaTrendAnalysis
$trendAnalysis | Format-Table -AutoSize
```

## 🔍 Quota Troubleshooting

### Common Quota Issues

#### Quota Not Applying
```powershell
# Function to troubleshoot quota application issues
function Test-QuotaApplication {
    param(
        [string]$Path
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
        # Test 2: Check if quota is configured
        try {
            $quota = Get-FsrmQuota -Path $Path -ErrorAction Stop
            $troubleshooting += [PSCustomObject]@{
                Test = "Quota Configuration"
                Result = "Pass"
                Details = "Quota configured: $($quota.Size) bytes"
            }
        } catch {
            $troubleshooting += [PSCustomObject]@{
                Test = "Quota Configuration"
                Result = "Fail"
                Details = "No quota configured for path"
            }
        }
        
        # Test 3: Check FSRM service status
        $fsrmService = Get-Service -Name Srmsvc
        $troubleshooting += [PSCustomObject]@{
            Test = "FSRM Service"
            Result = if ($fsrmService.Status -eq "Running") { "Pass" } else { "Fail" }
            Details = "FSRM service status: $($fsrmService.Status)"
        }
        
        # Test 4: Check NTFS permissions
        try {
            $acl = Get-Acl $Path
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
}

# Troubleshoot quota application
Test-QuotaApplication -Path "D:\Shares\HR\john.smith"
```

#### Quota Exceeded Issues
```powershell
# Function to troubleshoot quota exceeded issues
function Test-QuotaExceeded {
    param(
        [string]$UserName
    )
    
    # Find user's quota
    $userQuotas = Get-FsrmQuota | Where-Object {$_.Path -like "*$UserName*"}
    
    foreach ($quota in $userQuotas) {
        $usagePercent = ($quota.Used / $quota.Size) * 100
        
        if ($usagePercent -ge 100) {
            Write-Host "Quota exceeded for $UserName at $($quota.Path)"
            
            # Find large files that can be cleaned up
            $largeFiles = Get-ChildItem -Path $quota.Path -Recurse -File -ErrorAction SilentlyContinue | 
                Sort-Object Length -Descending | 
                Select-Object -First 10
            
            Write-Host "Large files in $($quota.Path):"
            $largeFiles | Select-Object Name, @{Name="SizeMB";Expression={[math]::Round($_.Length / 1MB, 2)}}
        }
    }
}

# Test quota exceeded scenarios
Test-QuotaExceeded -UserName "john.smith"
```

## 🖼️ Screenshots

### Quota Template Creation
![Quota Template](../screenshots/05-file-server/quota-template.png)

### Quota Configuration
![Quota Setup](../screenshots/05-file-server/quota-configuration.png)

### Quota Monitoring
![Quota Monitoring](../screenshots/05-file-server/quota-monitoring.png)

### Quota Report
![Quota Report](../screenshots/05-file-server/quota-report.png)

## 📋 Quota Configuration Summary

| Department | Quota Limit | Warning Threshold | Users | Status |
|------------|---------------|-------------------|--------|---------|
| HR | 2GB | 80% (1.6GB) | 2 | ✅ Configured |
| Sales | 3GB | 75% (2.25GB) | 2 | ✅ Configured |
| IT | 5GB | 70% (3.5GB) | 2 | ✅ Configured |
| Public | 1GB | 90% (900MB) | Shared | ✅ Configured |

### Key Features Implemented
- ✅ FSRM quota management installed and configured
- ✅ Departmental quota templates created
- ✅ HR users: 2GB quota with 80% warning
- ✅ Sales users: 3GB quota with 75% warning
- ✅ IT users: 5GB quota with 70% warning
- ✅ Public folder: 1GB quota with 90% warning
- ✅ Email notifications configured for quota thresholds
- ✅ Comprehensive monitoring and reporting procedures
- ✅ Automated cleanup and maintenance scripts

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
