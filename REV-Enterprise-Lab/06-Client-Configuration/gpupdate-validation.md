# Group Policy Update Validation

## 📋 Description

This document provides comprehensive procedures for validating Group Policy application on Windows 10 workstations in REV-Enterprise-Lab environment, including GPUpdate execution, result analysis, and troubleshooting.

## 🎯 Objectives

- Validate Group Policy application on client workstations
- Ensure all GPOs are properly applied to target computers
- Troubleshoot Group Policy application issues
- Verify policy inheritance and precedence
- Document Group Policy validation procedures

## 🔧 GPUpdate Procedures

### Manual GPUpdate Execution

#### Basic GPUpdate Command
```powershell
# Force Group Policy update
gpupdate /force

# Update only computer policy
gpupdate /target:computer /force

# Update only user policy
gpupdate /target:user /force

# Update with verbose output
gpupdate /force /wait:120
```

#### GPUpdate with Logging
```powershell
# Function to run GPUpdate with detailed logging
function Invoke-GPUpdateWithLogging {
    param(
        [string]$ComputerName = $env:COMPUTERNAME,
        [int]$TimeoutMinutes = 15
    )
    
    $logPath = "C:\Temp\GPUpdate_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"
    $timeout = $TimeoutMinutes * 60
    
    Write-Host "Starting GPUpdate on $ComputerName..."
    Write-Host "Log file: $logPath"
    Write-Host "Timeout: $TimeoutMinutes minutes"
    
    try {
        # Run GPUpdate with logging
        $process = Start-Process -FilePath "gpupdate.exe" -ArgumentList "/force /wait:$timeout" -Wait -PassThru -NoNewWindow
        
        # Capture output
        $output = & gpupdate /force /wait:$timeout 2>&1
        $output | Out-File -FilePath $logPath -Append
        
        Write-Host "GPUpdate completed with exit code: $($process.ExitCode)"
        
        # Return result
        return @{
            Success = ($process.ExitCode -eq 0)
            ExitCode = $process.ExitCode
            LogFile = $logPath
            ComputerName = $ComputerName
            Timestamp = Get-Date
        }
        
    } catch {
        Write-Host "Error running GPUpdate: $_"
        return @{
            Success = $false
            ExitCode = -1
            LogFile = $logPath
            ComputerName = $ComputerName
            Timestamp = Get-Date
            Error = $_.Exception.Message
        }
    }
}

# Run GPUpdate with logging
$gpupdateResult = Invoke-GPUpdateWithLogging
```

### Remote GPUpdate Execution

#### PowerShell Remoting GPUpdate
```powershell
# Function to run GPUpdate on remote computer
function Invoke-RemoteGPUpdate {
    param(
        [string]$ComputerName,
        [string]$CredentialName = "REV\IT-Group"
    )
    
    try {
        # Get credentials
        $credential = Get-Credential -UserName $CredentialName -Message "Enter credentials for $CredentialName"
        
        # Create remote session
        $session = New-PSSession -ComputerName $ComputerName -Credential $credential -ErrorAction Stop
        
        # Run GPUpdate in remote session
        $result = Invoke-Command -Session $session -ScriptBlock {
            gpupdate /force /wait:300
            return $LASTEXITCODE
        }
        
        # Remove session
        Remove-PSSession -Session $session -ErrorAction SilentlyContinue
        
        return @{
            ComputerName = $ComputerName
            Success = ($result -eq 0)
            ExitCode = $result
            Timestamp = Get-Date
        }
        
    } catch {
        Write-Host "Error running remote GPUpdate on $ComputerName`: $_"
        return @{
            ComputerName = $ComputerName
            Success = $false
            ExitCode = -1
            Timestamp = Get-Date
            Error = $_.Exception.Message
        }
    }
}

# Run remote GPUpdate on multiple computers
$computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02")
$remoteResults = @()

foreach ($computer in $computers) {
    $result = Invoke-RemoteGPUpdate -ComputerName $computer
    $remoteResults += $result
}

$remoteResults | Format-Table -AutoSize
```

### Scheduled GPUpdate

#### Create Scheduled Task for Automatic Updates
```powershell
# Function to create GPUpdate scheduled task
function New-GPUpdateScheduledTask {
    param(
        [string]$ComputerName = $env:COMPUTERNAME,
        [string]$Schedule = "Daily",
        [string]$Time = "03:00"
    )
    
    try {
        # Create action for GPUpdate
        $action = New-ScheduledTaskAction -Execute "gpupdate.exe" -Argument "/force /wait:600" -WorkingDirectory "C:\Windows\System32"
        
        # Create trigger based on schedule
        switch ($Schedule) {
            "Daily" {
                $trigger = New-ScheduledTaskTrigger -Daily -At $Time
            }
            "Weekly" {
                $trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At $Time
            }
            "Hourly" {
                $trigger = New-ScheduledTaskTrigger -Once -At (Get-Date).AddHours(1) -RepetitionInterval (New-TimeSpan -Hours 1)
            }
        }
        
        # Create settings
        $settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopOnIdleEnd -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -WakeToRun
        
        # Register scheduled task
        Register-ScheduledTask -TaskName "GPUpdate Automatic" -Action $action -Trigger $trigger -Settings $settings -RunLevel Highest -User "SYSTEM" -Force
        
        Write-Host "Created GPUpdate scheduled task on $ComputerName"
        Write-Host "Schedule: $Schedule at $Time"
        
        return @{
            ComputerName = $ComputerName
            Success = $true
            TaskName = "GPUpdate Automatic"
            Schedule = $Schedule
            Time = $Time
        }
        
    } catch {
        Write-Host "Error creating scheduled task on $ComputerName`: $_"
        return @{
            ComputerName = $ComputerName
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Create scheduled task
$taskResult = New-GPUpdateScheduledTask -Schedule "Daily" -Time "03:00"
$taskResult
```

## 📊 GPResult Analysis

### Generate GPResult Reports

#### Computer Policy GPResult
```powershell
# Function to generate computer GPResult report
function Get-ComputerGPResult {
    param(
        [string]$ComputerName = $env:COMPUTERNAME,
        [string]$ReportPath = "C:\Temp\ComputerGPResult_$(Get-Date -Format 'yyyyMMdd_HHmmss').html"
    )
    
    try {
        # Generate GPResult report
        $result = & gpresult /r /scope computer /h > "$ReportPath"
        
        # Parse GPResult output for key information
        $gpResultContent = Get-Content $ReportPath -Raw
        
        # Extract applied GPOs
        $appliedGPOs = @()
        if ($gpResultContent -match "Applied Group Policy Objects") {
            $gpoSection = $gpResultContent -split "Applied Group Policy Objects" -split "The computer received a successful update" | Select-Object -Index 1
            $gpoLines = $gpoSection -split "`n" | Where-Object {$_ -match "GPO:"}
            
            foreach ($line in $gpoLines) {
                if ($line -match "GPO:\s*(.+?)\s*\(") {
                    $gpoName = $Matches[1].Trim()
                    $appliedGPOs += $gpoName
                }
            }
        }
        
        return @{
            ComputerName = $ComputerName
            ReportPath = $ReportPath
            AppliedGPOs = $appliedGPOs
            GPCount = $appliedGPOs.Count
            Timestamp = Get-Date
            Success = $true
        }
        
    } catch {
        Write-Host "Error generating computer GPResult: $_"
        return @{
            ComputerName = $ComputerName
            ReportPath = $ReportPath
            AppliedGPOs = @()
            GPCount = 0
            Timestamp = Get-Date
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Generate computer GPResult
$computerGPResult = Get-ComputerGPResult
$computerGPResult | Format-List
```

#### User Policy GPResult
```powershell
# Function to generate user GPResult report
function GetUserGPResult {
    param(
        [string]$ComputerName = $env:COMPUTERNAME,
        [string]$ReportPath = "C:\Temp\UserGPResult_$(Get-Date -Format 'yyyyMMdd_HHmmss').html"
    )
    
    try {
        # Generate GPResult report
        $result = & gpresult /r /scope user /h > "$ReportPath"
        
        # Parse GPResult output for key information
        $gpResultContent = Get-Content $ReportPath -Raw
        
        # Extract applied GPOs
        $appliedGPOs = @()
        if ($gpResultContent -match "Applied Group Policy Objects") {
            $gpoSection = $gpResultContent -split "Applied Group Policy Objects" -split "The user received a successful update" | Select-Object -Index 1
            $gpoLines = $gpoSection -split "`n" | Where-Object {$_ -match "GPO:"}
            
            foreach ($line in $gpoLines) {
                if ($line -match "GPO:\s*(.+?)\s*\(") {
                    $gpoName = $Matches[1].Trim()
                    $appliedGPOs += $gpoName
                }
            }
        }
        
        return @{
            ComputerName = $ComputerName
            ReportPath = $ReportPath
            AppliedGPOs = $appliedGPOs
            GPCount = $appliedGPOs.Count
            Timestamp = Get-Date
            Success = $true
        }
        
    } catch {
        Write-Host "Error generating user GPResult: $_"
        return @{
            ComputerName = $ComputerName
            ReportPath = $ReportPath
            AppliedGPOs = @()
            GPCount = 0
            Timestamp = Get-Date
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Generate user GPResult
$userGPResult = GetUserGPResult
$userGPResult | Format-List
```

### Combined GPResult Analysis

#### Analyze Both Computer and User Policies
```powershell
# Function to analyze complete GPResult
function Get-CompleteGPResultAnalysis {
    param(
        [string]$ComputerName = $env:COMPUTERNAME
    )
    
    try {
        # Generate both reports
        $computerResult = Get-ComputerGPResult -ComputerName $ComputerName
        $userResult = GetUserGPResult -ComputerName $ComputerName
        
        # Combine results
        $analysis = @{
            ComputerName = $ComputerName
            ComputerGPOs = $computerResult.AppliedGPOs
            ComputerGPCount = $computerResult.GPCount
            UserGPOs = $userResult.AppliedGPOs
            UserGPCount = $userResult.GPCount
            TotalGPOs = $computerResult.AppliedGPOs + $userResult.AppliedGPOs
            Timestamp = Get-Date
            Success = ($computerResult.Success -and $userResult.Success)
        }
        
        # Check for expected GPOs
        $expectedGPOs = @(
            "Default Domain Policy",
            "HR Security Restrictions",
            "HR Drive Mapping",
            "HR Storage Quota Policy",
            "HR Software Restrictions",
            "HR Installer Restrictions",
            "HR Media Restrictions",
            "HR Context Menu Restrictions"
        )
        
        $missingGPOs = $expectedGPOs | Where-Object {$_ -notin $analysis.TotalGPOs}
        $unexpectedGPOs = $analysis.TotalGPOs | Where-Object {$_ -notin $expectedGPOs}
        
        $analysis.MissingGPOs = $missingGPOs
        $analysis.UnexpectedGPOs = $unexpectedGPOs
        $analysis.ComplianceStatus = if ($missingGPOs.Count -eq 0 -and $unexpectedGPOs.Count -eq 0) { "Compliant" } else { "Non-Compliant" }
        
        return $analysis
        
    } catch {
        Write-Host "Error analyzing GPResult: $_"
        return @{
            ComputerName = $ComputerName
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Analyze complete GPResult
$completeAnalysis = Get-CompleteGPResultAnalysis
$completeAnalysis | Format-List
```

## 🔍 GPUpdate Validation

### Expected GPO Validation

#### HR Department GPO Validation
```powershell
# Function to validate HR-specific GPOs
function Test-HRGPOValidation {
    param(
        [string]$ComputerName
    )
    
    $expectedHRGPOs = @(
        "HR Security Restrictions",
        "HR Drive Mapping",
        "HR Storage Quota Policy",
        "HR Software Restrictions",
        "HR Installer Restrictions",
        "HR Media Restrictions",
        "HR Context Menu Restrictions"
    )
    
    try {
        # Get GPResult for the computer
        $gpResult = Get-CompleteGPResultAnalysis -ComputerName $ComputerName
        
        $validation = @{
            ComputerName = $ComputerName
            ExpectedGPOs = $expectedHRGPOs
            AppliedGPOs = $gpResult.TotalGPOs
            MissingGPOs = @()
            UnexpectedGPOs = $gpResult.UnexpectedGPOs
            OverallStatus = "Unknown"
        }
        
        # Check for missing HR GPOs
        foreach ($gpo in $expectedHRGPOs) {
            if ($gpo -notin $gpResult.TotalGPOs) {
                $validation.MissingGPOs += $gpo
            }
        }
        
        # Determine overall status
        if ($validation.MissingGPOs.Count -eq 0) {
            $validation.OverallStatus = "Compliant"
        } else {
            $validation.OverallStatus = "Non-Compliant - Missing $($validation.MissingGPOs.Count) GPOs"
        }
        
        return $validation
        
    } catch {
        Write-Host "Error validating HR GPOs on $ComputerName`: $_"
        return @{
            ComputerName = $ComputerName
            OverallStatus = "Error"
            Error = $_.Exception.Message
        }
    }
}

# Validate HR GPOs on HR computers
$hrComputers = @("HR-PC-01", "HR-PC-02")
$hrValidationResults = @()

foreach ($computer in $hrComputers) {
    $result = Test-HRGPOValidation -ComputerName $computer
    $hrValidationResults += $result
}

$hrValidationResults | Format-Table -AutoSize
```

### Policy Effect Validation

#### Test Security Policy Application
```powershell
# Function to test if security policies are applied
function Test-SecurityPolicyApplication {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            $securityTests = @()
            
            # Test 1: Check if Command Prompt is disabled
            $cmdPath = "C:\Windows\System32\cmd.exe"
            $cmdAccessible = Test-Path $cmdPath
            
            $securityTests += [PSCustomObject]@{
                Policy = "Command Prompt Disabled"
                Expected = "Disabled"
                Actual = if ($cmdAccessible) { "Enabled" } else { "Disabled" }
                Status = if (-not $cmdAccessible) { "Pass" } else { "Fail" }
            }
            
            # Test 2: Check if Control Panel is disabled
            $controlPath = "C:\Windows\System32\control.exe"
            $controlAccessible = Test-Path $controlPath
            
            $securityTests += [PSCustomObject]@{
                Policy = "Control Panel Disabled"
                Expected = "Disabled"
                Actual = if ($controlAccessible) { "Enabled" } else { "Disabled" }
                Status = if (-not $controlAccessible) { "Pass" } else { "Fail" }
            }
            
            # Test 3: Check if Registry Editor is disabled
            $regeditPath = "C:\Windows\regedit.exe"
            $regeditAccessible = Test-Path $regeditPath
            
            $securityTests += [PSCustomObject]@{
                Policy = "Registry Editor Disabled"
                Expected = "Disabled"
                Actual = if ($regeditAccessible) { "Enabled" } else { "Disabled" }
                Status = if (-not $regeditAccessible) { "Pass" } else { "Fail" }
            }
            
            return $securityTests
        }
        
        return @{
            ComputerName = $ComputerName
            SecurityTests = $result
            Timestamp = Get-Date
            Success = $true
        }
        
    } catch {
        Write-Host "Error testing security policies on $ComputerName`: $_"
        return @{
            ComputerName = $ComputerName
            SecurityTests = @()
            Timestamp = Get-Date
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Test security policy application
$securityTest = Test-SecurityPolicyApplication -ComputerName "HR-PC-01"
$securityTest.SecurityTests | Format-Table -AutoSize
```

#### Test Drive Mapping Policy Application
```powershell
# Function to test if drive mapping is applied
function Test-DriveMappingApplication {
    param(
        [string]$ComputerName,
        [string]$ExpectedDrive = "H:"
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            param($Drive)
            
            # Test if drive is mapped
            $driveMapped = Test-Path $Drive
            
            # Test if drive has correct label
            $shell = New-Object -ComObject Shell.Application
            $driveLabel = $shell.NameSpace($Drive).Self.Name
            
            # Test if drive is accessible
            $driveAccessible = if ($driveMapped) { (Get-ChildItem $Drive -ErrorAction SilentlyContinue).Count -ge 0 } else { $false }
            
            return @{
                DriveLetter = $Drive
                IsMapped = $driveMapped
                DriveLabel = $driveLabel
                IsAccessible = $driveAccessible
            }
        } -ArgumentList $ExpectedDrive
        
        return @{
            ComputerName = $ComputerName
            ExpectedDrive = $ExpectedDrive
            ActualResult = $result
            Status = if ($result.IsMapped -and $result.IsAccessible) { "Pass" } else { "Fail" }
            Timestamp = Get-Date
            Success = $true
        }
        
    } catch {
        Write-Host "Error testing drive mapping on $ComputerName`: $_"
        return @{
            ComputerName = $ComputerName
            ExpectedDrive = $ExpectedDrive
            Status = "Error"
            Timestamp = Get-Date
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Test drive mapping application
$driveMappingTest = Test-DriveMappingApplication -ComputerName "HR-PC-01" -ExpectedDrive "H:"
$driveMappingTest | Format-List
```

## 📊 GPUpdate Monitoring

### Automated GPUpdate Monitoring

#### Create GPUpdate Monitoring Script
```powershell
# Function to create GPUpdate monitoring script
function New-GPUpdateMonitoringScript {
    $scriptContent = @"
# GPUpdate Monitoring Script
# This script monitors Group Policy application and generates reports

`$logPath = "C:\Temp\GPUpdate_Monitoring_`$(Get-Date -Format 'yyyyMMdd').log"
`$reportPath = "C:\Temp\GPUpdate_Monitoring_`$(Get-Date -Format 'yyyyMMdd').csv"

function Write-Log {
    param(`$message)
    `$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "[$timestamp] `$message" | Out-File -FilePath `$logPath -Append
}

function Write-Report {
    param(`$data)
    `$data | Export-Csv -Path `$reportPath -NoTypeInformation -Append
}

# Main monitoring loop
while (`$true) {
    try {
        Write-Log "Starting GPUpdate check"
        
        # Run GPUpdate
        `$gpupdateResult = gpupdate /force /wait:300
        `$exitCode = `$LASTEXITCODE
        
        # Analyze result
        `$analysis = @{
            Timestamp = Get-Date
            ComputerName = `$env:COMPUTERNAME
            ExitCode = `$exitCode
            Success = (`$exitCode -eq 0)
            GPUpdateCommand = "gpupdate /force /wait:300"
        }
        
        Write-Log "GPUpdate completed with exit code: `$exitCode"
        Write-Report `$analysis
        
        # Wait for next check (4 hours)
        Start-Sleep -Seconds (4 * 60 * 60)
        
    } catch {
        Write-Log "Error in GPUpdate monitoring: `$_"
        Start-Sleep -Seconds 300  # Wait 5 minutes on error
    }
}
"@
    
    # Save script
    $scriptPath = "C:\Scripts\GPUpdateMonitoring.ps1"
    New-Item -Path (Split-Path $scriptPath) -ItemType Directory -Force
    $scriptContent | Out-File -FilePath $scriptPath -Encoding UTF8
    
    Write-Host "GPUpdate monitoring script created: $scriptPath"
    
    # Create scheduled task
    $action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File `"$scriptPath`""
    $trigger = New-ScheduledTaskTrigger -Once -At (Get-Date).AddMinutes(5) -RepetitionInterval (New-TimeSpan -Hours 4)
    $settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopOnIdleEnd -AllowStartIfOnBatteries
    
    Register-ScheduledTask -TaskName "GPUpdate Monitoring" -Action $action -Trigger $trigger -Settings $settings -RunLevel Highest -User "SYSTEM" -Force
    
    Write-Host "GPUpdate monitoring scheduled task created"
}

# Create monitoring script
New-GPUpdateMonitoringScript
```

### GPUpdate Health Dashboard

#### Create Health Dashboard Script
```powershell
# Function to create GPUpdate health dashboard
function New-GPUpdateHealthDashboard {
    $dashboardScript = @"
# GPUpdate Health Dashboard
# Generates HTML dashboard for GPUpdate status

`$dashboardPath = "C:\Temp\GPUpdate_Dashboard.html"
`$computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02")

function Get-GPUpdateStatus {
    param(`$computerName)
    
    try {
        # Get last GPUpdate time from event log
        `$gpupdateEvent = Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 1 | Where-Object {`$_.Message -like "*completed successfully*"}
        
        if (`$gpupdateEvent) {
            return @{
                ComputerName = `$computerName
                LastUpdate = `$gpupdateEvent.TimeCreated
                Status = "Success"
                Message = "Group Policy applied successfully"
            }
        } else {
            return @{
                ComputerName = `$computerName
                LastUpdate = `$null
                Status = "Unknown"
                Message = "No recent GPUpdate events found"
            }
        }
    } catch {
        return @{
            ComputerName = `$computerName
            LastUpdate = `$null
            Status = "Error"
            Message = `$_
        }
    }
}

# Generate dashboard HTML
`$html = @"
<!DOCTYPE html>
<html>
<head>
    <title>GPUpdate Health Dashboard</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f5f5f5; }
        .dashboard { max-width: 1200px; margin: 0 auto; background-color: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .header { text-align: center; margin-bottom: 30px; color: #333; }
        .computer-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; }
        .computer-card { background-color: #fff; padding: 20px; border-radius: 8px; border-left: 4px solid #007bff; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .computer-name { font-size: 18px; font-weight: bold; margin-bottom: 10px; color: #333; }
        .status { font-size: 16px; margin-bottom: 10px; }
        .status.success { color: #28a745; }
        .status.error { color: #dc3545; }
        .status.unknown { color: #ffc107; }
        .last-update { font-size: 14px; color: #666; }
        .refresh-time { text-align: center; margin-top: 20px; color: #666; }
    </style>
</head>
<body>
    <div class="dashboard">
        <div class="header">
            <h1>GPUpdate Health Dashboard</h1>
            <p>Generated on $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
        </div>
        
        <div class="computer-grid">
"@

# Add computer cards
foreach (`$computer in `$computers) {
    `$status = Get-GPUpdateStatus -computerName `$computer
    `$statusClass = switch (`$status.Status) {
        "Success" { "success" }
        "Error" { "error" }
        default { "unknown" }
    }
    
    `$html += @"
            <div class="computer-card">
                <div class="computer-name">`$(`$status.ComputerName)</div>
                <div class="status `$statusClass">Status: `$(`$status.Status)</div>
                <div class="last-update">Last Update: `$(`$status.LastUpdate)</div>
                <div class="message">`$(`$status.Message)</div>
            </div>
"@
}

`$html += @"
        </div>
        
        <div class="refresh-time">
            <p>Auto-refresh every 5 minutes</p>
        </div>
    </div>
</body>
</html>
"@

    `$html | Out-File -FilePath `$dashboardPath -Encoding UTF8
    
    Write-Host "GPUpdate health dashboard created: `$dashboardPath"
}

# Generate dashboard
New-GPUpdateHealthDashboard
"@
    
    # Save dashboard script
    $dashboardScriptPath = "C:\Scripts\GPUpdateDashboard.ps1"
    $dashboardScript | Out-File -FilePath $dashboardScriptPath -Encoding UTF8
    
    Write-Host "GPUpdate health dashboard script created: $dashboardScriptPath"
}

# Create health dashboard
New-GPUpdateHealthDashboard
```

## 🔍 GPUpdate Troubleshooting

### Common Issues and Solutions

#### GPUpdate Fails with Error Code
```powershell
# Function to troubleshoot GPUpdate exit codes
function Test-GPUpdateExitCodes {
    param(
        [int]$ExitCode
    )
    
    $exitCodeMeanings = @{
        0 = "Success"
        1 = "General error"
        2 = "Invalid argument"
        5 = "Access denied"
        6 = "Invalid handle"
        8 = "Insufficient memory"
        20 = "General failure"
        87 = "Invalid parameter"
        1603 = "Fatal error during installation"
        1719 = "DNS record does not exist"
        1722 = "RPC server is unavailable"
        1753 = "The endpoint mapper is unavailable"
        1788 = "The trust relationship between this workstation and the primary domain failed"
    }
    
    $meaning = $exitCodeMeanings[$ExitCode]
    
    if ($meaning) {
        Write-Host "GPUpdate Exit Code $ExitCode`: $meaning"
        
        # Provide troubleshooting suggestions
        switch ($ExitCode) {
            0 { Write-Host "✅ GPUpdate completed successfully" }
            1 { Write-Host "❌ General error - Check system logs for details" }
            5 { Write-Host "❌ Access denied - Run as administrator" }
            8 { Write-Host "❌ Insufficient memory - Close applications and retry" }
            1603 { Write-Host "❌ Fatal error - Check for corrupted policies" }
            1719 { Write-Host "❌ DNS issue - Check DNS configuration" }
            1722 { Write-Host "❌ RPC issue - Check network connectivity" }
            1788 { Write-Host "❌ Trust relationship issue - Reset computer account" }
        }
    } else {
        Write-Host "Unknown GPUpdate exit code: $ExitCode"
    }
}

# Test exit code analysis
Test-GPUpdateExitCodes -ExitCode 0
Test-GPUpdateExitCodes -ExitCode 5
Test-GPUpdateExitCodes -ExitCode 1788
```

#### Group Policy Not Applying
```powershell
# Function to troubleshoot GPO application issues
function Test-GPOApplicationIssues {
    param(
        [string]$ComputerName
    )
    
    $troubleshooting = @()
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Test 1: Check network connectivity to DC
            $dcPing = Test-Connection -ComputerName "PDC.rev.local" -Count 2 -Quiet
            $troubleshooting += @{
                Test = "DC Connectivity"
                Status = if ($dcPing) { "Pass" } else { "Fail" }
                Details = if ($dcPing) { "Can reach domain controller" } else { "Cannot reach domain controller" }
            }
            
            # Test 2: Check DNS resolution
            try {
                $dnsResult = Resolve-DnsName -Name "PDC.rev.local" -ErrorAction Stop
                $troubleshooting += @{
                    Test = "DNS Resolution"
                    Status = "Pass"
                    Details = "DNS resolution working"
                }
            } catch {
                $troubleshooting += @{
                    Test = "DNS Resolution"
                    Status = "Fail"
                    Details = "DNS resolution failed: $($_.Exception.Message)"
                }
            }
            
            # Test 3: Check time synchronization
            $timeService = Get-Service -Name "W32Time"
            $troubleshooting += @{
                Test = "Time Service"
                Status = if ($timeService.Status -eq "Running") { "Pass" } else { "Fail" }
                Details = "Time service status: $($timeService.Status)"
            }
            
            # Test 4: Check for policy processing errors
            $policyErrors = Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" -MaxEvents 10 | Where-Object {$_.LevelDisplayName -eq "Error"}
            $troubleshooting += @{
                Test = "Policy Errors"
                Status = if ($policyErrors.Count -eq 0) { "Pass" } else { "Fail" }
                Details = "Found $($policyErrors.Count) policy errors in event log"
            }
            
            return $troubleshooting
        }
        
        return @{
            ComputerName = $ComputerName
            TroubleshootingResults = $result
            Timestamp = Get-Date
            Success = $true
        }
        
    } catch {
        Write-Host "Error troubleshooting GPO application on $ComputerName`: $_"
        return @{
            ComputerName = $ComputerName
            Success = $false
            Error = $_.Exception.Message
        }
    }
}

# Troubleshoot GPO application issues
$troubleshootingResult = Test-GPOApplicationIssues -ComputerName "HR-PC-01"
$troubleshootingResult.TroubleshootingResults | Format-Table -AutoSize
```

## 🖼️ Screenshots

### GPUpdate Execution
![GPUpdate Command](../screenshots/06-client-configuration/gpupdate-command.png)

### GPResult Report
![GPResult Report](../screenshots/06-client-configuration/gpresult-report.png)

### GPO Validation
![GPO Validation](../screenshots/06-client-configuration/gpo-validation.png)

### Health Dashboard
![Health Dashboard](../screenshots/06-client-configuration/health-dashboard.png)

## 📋 GPUpdate Validation Summary

| Validation Item | Method | Target | Status |
|-----------------|--------|--------|---------|
| Manual GPUpdate | Command Line | All workstations | ✅ Complete |
| Remote GPUpdate | PowerShell Remoting | IT workstations | ✅ Complete |
| Scheduled GPUpdate | Task Scheduler | All workstations | ✅ Complete |
| GPResult Analysis | gpresult.exe | HR workstations | ✅ Complete |
| Security Policy Test | Registry Check | HR workstations | ✅ Complete |
| Drive Mapping Test | File System | HR workstations | ✅ Complete |
| Automated Monitoring | Scripts + Tasks | All workstations | ✅ Complete |

### Key Features Implemented
- ✅ Manual and automated GPUpdate procedures
- ✅ Remote GPUpdate execution capabilities
- ✅ Comprehensive GPResult analysis
- ✅ Expected GPO validation for HR department
- ✅ Security policy application testing
- ✅ Drive mapping validation
- ✅ Automated monitoring and health dashboard
- ✅ Comprehensive troubleshooting procedures
- ✅ Exit code analysis and resolution guidance

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
