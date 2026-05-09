# Endpoint Restrictions Configuration

## 📋 Description

This document provides comprehensive procedures for configuring endpoint security restrictions in REV-Enterprise-Lab environment, including application restrictions, system hardening, and user access controls.

## 🎯 Objectives

- Implement comprehensive endpoint security restrictions
- Configure application and system limitations
- Establish user access control policies
- Implement system hardening measures
- Ensure compliance with security best practices

## 🔧 Endpoint Security Configuration

### Application Restrictions

#### Create Application Restrictions GPO
```powershell
# Create Application Restrictions GPO for HR
New-GPO -Name "HR Application Restrictions" -Comment "Comprehensive application restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Application Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure application restrictions
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "DisallowRun" -Type DWORD -Value 1

# Add disallowed applications
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "1" -Type String -Value "cmd.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "2" -Type String -Value "powershell.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "3" -Type String -Value "mspaint.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "4" -Type String -Value "notepad.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "5" -Type String -Value "regedit.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "6" -Type String -Value "taskmgr.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "7" -Type String -Value "gpedit.msc"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "8" -Type String -Value "services.msc"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "9" -Type String -Value "compmgmt.msc"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "10" -Type String -Value "devmgmt.msc"
```

#### Configure Allowed Applications Only
```powershell
# Configure allowed applications only
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "RestrictRun" -Type DWORD -Value 1

# Add allowed applications
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "1" -Type String -Value "WINWORD.EXE"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "2" -Type String -Value "EXCEL.EXE"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "3" -Type String -Value "POWERPNT.EXE"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "4" -Type String -Value "OUTLOOK.EXE"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "5" -Type String -Value "iexplore.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "6" -Type String -Value "chrome.exe"
Set-GPRegistryValue -Name "HR Application Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\RestrictRun" -ValueName "7" -Type String -Value "AcroRd32.exe"
```

### System Restrictions

#### Configure System Access Restrictions
```powershell
# Create System Restrictions GPO
New-GPO -Name "HR System Restrictions" -Comment "System access restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR System Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Command Prompt
Set-GPRegistryValue -Name "HR System Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableCMD" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR System Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableCommandPrompt" -Type DWORD -Value 1

# Disable Registry Editor
Set-GPRegistryValue -Name "HR System Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" -ValueName "DisableRegistryTools" -Type DWORD -Value 1

# Disable Task Manager
Set-GPRegistryValue -Name "HR System Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System" -ValueName "DisableTaskMgr" -Type DWORD -Value 1

# Disable System Restore
Set-GPRegistryValue -Name "HR System Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\SystemRestore" -ValueName "DisableSR" -Type DWORD -Value 1

# Disable Windows Update
Set-GPRegistryValue -Name "HR System Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -ValueName "NoAutoUpdate" -Type DWORD -Value 1
```

#### Configure Control Panel Restrictions
```powershell
# Create Control Panel Restrictions GPO
New-GPO -Name "HR Control Panel Restrictions" -Comment "Control Panel restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Control Panel Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Control Panel
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoControlPanel" -Type DWORD -Value 1

# Disable specific Control Panel items
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "DisallowCpl" -Type DWORD -Value 1

# Add disallowed Control Panel items
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "1" -Type String -Value "system.cpl"
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "2" -Type String -Value "userpasswords.cpl"
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "3" -Type String -Value "appwiz.cpl"
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "4" -Type String -Value "ncpa.cpl"
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "5" -Type String -Value "firewall.cpl"
```

### Network Restrictions

#### Configure Network Access Restrictions
```powershell
# Create Network Restrictions GPO
New-GPO -Name "HR Network Restrictions" -Comment "Network access restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Network Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Internet access
Set-GPRegistryValue -Name "HR Network Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -ValueName "ProxyEnable" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Network Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -ValueName "ProxyServer" -Type String -Value "127.0.0.1:8080"

# Disable Network Connections
Set-GPRegistryValue -Name "HR Network Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Network Connections" -ValueName "NoConnectionsPage" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Network Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Network Connections" -ValueName "NoNetSetup" -Type DWORD -Value 1

# Disable Network Setup Wizard
Set-GPRegistryValue -Name "HR Network Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Network Connections" -ValueName "NoNetSetupWizard" -Type DWORD -Value 1

# Disable Network Map
Set-GPRegistryValue -Name "HR Network Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\Network Connections" -ValueName "NoNetworkMap" -Type DWORD -Value 1
```

## 🔍 Advanced Endpoint Security

### Windows Defender Configuration

#### Configure Windows Defender Policies
```powershell
# Create Windows Defender GPO
New-GPO -Name "HR Windows Defender" -Comment "Windows Defender configuration for HR department"

# Link to HR OU
New-GPLink -Name "HR Windows Defender" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Enable Windows Defender
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" -ValueName "DisableAntiSpyware" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" -ValueName "DisableRoutinelyTakingAction" -Type DWORD -Value 0

# Configure real-time protection
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -ValueName "DisableRealtimeMonitoring" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -ValueName "DisableIOAVProtection" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" -ValueName "DisableScriptScanning" -Type DWORD -Value 0

# Configure scan settings
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Scan" -ValueName "DisableRemovableDriveScanning" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Scan" -ValueName "DisableScanningNetworkFiles" -Type DWORD -Value 1

# Configure update settings
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Signature Updates" -ValueName "DisableSignatureUpdate" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Defender" -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Signature Updates" -ValueName "UpdateOnScheduledRunOnly" -Type DWORD -Value 0
```

### Windows Firewall Configuration

#### Configure Windows Firewall Policies
```powershell
# Create Windows Firewall GPO
New-GPO -Name "HR Windows Firewall" -Comment "Windows Firewall configuration for HR department"

# Link to HR OU
New-GPLink -Name "HR Windows Firewall" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Enable firewall for all profiles
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile" -ValueName "EnableFirewall" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile" -ValueName "EnableFirewall" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile" -ValueName "EnableFirewall" -Type DWORD -Value 1

# Configure default inbound action
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile" -ValueName "DefaultInboundAction" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile" -ValueName "DefaultInboundAction" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile" -ValueName "DefaultInboundAction" -Type DWORD -Value 1

# Configure default outbound action
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile" -ValueName "DefaultOutboundAction" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile" -ValueName "DefaultOutboundAction" -Type DWORD -Value 0
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile" -ValueName "DefaultOutboundAction" -Type DWORD -Value 0

# Disable notifications
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile" -ValueName "DisableNotifications" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\StandardProfile" -ValueName "DisableNotifications" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Windows Firewall" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile" -ValueName "DisableNotifications" -Type DWORD -Value 1
```

## 📊 Endpoint Security Monitoring

### Application Monitoring
```powershell
# Function to monitor running applications
function Get-RunningApplications {
    $applications = Get-Process | Select-Object ProcessName, Path, Company, ProductVersion, StartTime, CPU, WorkingSet64
    
    $applicationReport = @()
    
    foreach ($app in $applications) {
        $applicationReport += [PSCustomObject]@{
            ProcessName = $app.ProcessName
            ExecutablePath = $app.Path
            Company = $app.Company
            Version = $app.ProductVersion
            StartTime = $app.StartTime
            CPUUsage = $app.CPU
            MemoryUsageMB = [math]::Round($app.WorkingSet64 / 1MB, 2)
            IsAllowed = $true  # Would need to check against allowed list
        }
    }
    
    return $applicationReport
}

# Get running applications
$runningApps = Get-RunningApplications
$runningApps | Sort-Object MemoryUsageMB -Descending | Select-Object -First 20 | Format-Table -AutoSize
```

### Security Policy Compliance Monitoring
```powershell
# Function to check endpoint security compliance
function Test-EndpointSecurityCompliance {
    $complianceResults = @()
    
    # Check 1: Application restrictions
    $disallowedApps = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "DisallowRun" -ErrorAction SilentlyContinue
    $complianceResults += [PSCustomObject]@{
        Test = "Application Restrictions"
        Status = if ($disallowedApps.DisallowRun -eq 1) { "Enabled" } else { "Disabled" }
        Details = "DisallowRun registry value: $($disallowedApps.DisallowRun)"
    }
    
    # Check 2: System restrictions
    $disableCMD = Get-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\System" -Name "DisableCMD" -ErrorAction SilentlyContinue
    $complianceResults += [PSCustomObject]@{
        Test = "Command Prompt Disabled"
        Status = if ($disableCMD.DisableCMD -eq 1) { "Enabled" } else { "Disabled" }
        Details = "DisableCMD registry value: $($disableCMD.DisableCMD)"
    }
    
    # Check 3: Registry Editor disabled
    $disableRegEdit = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "DisableRegistryTools" -ErrorAction SilentlyContinue
    $complianceResults += [PSCustomObject]@{
        Test = "Registry Editor Disabled"
        Status = if ($disableRegEdit.DisableRegistryTools -eq 1) { "Enabled" } else { "Disabled" }
        Details = "DisableRegistryTools registry value: $($disableRegEdit.DisableRegistryTools)"
    }
    
    # Check 4: Control Panel disabled
    $disableControlPanel = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoControlPanel" -ErrorAction SilentlyContinue
    $complianceResults += [PSCustomObject]@{
        Test = "Control Panel Disabled"
        Status = if ($disableControlPanel.NoControlPanel -eq 1) { "Enabled" } else { "Disabled" }
        Details = "NoControlPanel registry value: $($disableControlPanel.NoControlPanel)"
    }
    
    # Check 5: Windows Defender status
    $defenderStatus = Get-MpComputerStatus
    $complianceResults += [PSCustomObject]@{
        Test = "Windows Defender Status"
        Status = if ($defenderStatus.RealTimeProtectionEnabled) { "Enabled" } else { "Disabled" }
        Details = "Real-time protection: $($defenderStatus.RealTimeProtectionEnabled)"
    }
    
    # Check 6: Windows Firewall status
    $firewallStatus = Get-NetFirewallProfile -All
    $domainFirewall = $firewallStatus | Where-Object {$_.Name -eq "Domain"}
    $complianceResults += [PSCustomObject]@{
        Test = "Windows Firewall Status"
        Status = if ($domainFirewall.Enabled) { "Enabled" } else { "Disabled" }
        Details = "Domain firewall enabled: $($domainFirewall.Enabled)"
    }
    
    return $complianceResults
}

# Test endpoint security compliance
$complianceResults = Test-EndpointSecurityCompliance
$complianceResults | Format-Table -AutoSize
```

## 🔍 Endpoint Security Troubleshooting

### Common Issues and Solutions

#### Application Restrictions Not Working
```powershell
# Function to troubleshoot application restrictions
function Test-ApplicationRestrictions {
    $troubleshooting = @()
    
    # Test 1: Check if GPO exists
    $gpo = Get-GPO -Name "HR Application Restrictions" -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "GPO Existence"
        Status = if ($gpo) { "Pass" } else { "Fail" }
        Details = if ($gpo) { "Application restrictions GPO found" } else { "Application restrictions GPO not found" }
    }
    
    if ($gpo) {
        # Test 2: Check if GPO is linked
        $links = Get-GPLink -Guid $gpo.Id
        $hrLink = $links | Where-Object {$_.Target -eq "OU=Human Resources,OU=Departments,DC=rev,DC=local"}
        
        $troubleshooting += [PSCustomObject]@{
            Test = "GPO Linking"
            Status = if ($hrLink) { "Pass" } else { "Fail" }
            Details = if ($hrLink) { "GPO linked to HR OU" } else { "GPO not linked to HR OU" }
        }
        
        # Test 3: Check if registry settings are applied
        $disallowedApps = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "DisallowRun" -ErrorAction SilentlyContinue
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Registry Settings"
            Status = if ($disallowedApps) { "Applied" } else { "Not Applied" }
            Details = if ($disallowedApps) { "DisallowRun registry value found" } else { "DisallowRun registry value not found" }
        }
        
        # Test 4: Check if specific applications are blocked
        $blockedApps = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ErrorAction SilentlyContinue
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Blocked Applications"
            Status = if ($blockedApps) { "Configured" } else { "Not Configured" }
            Details = if ($blockedApps) { "Blocked applications found" } else { "No blocked applications found" }
        }
    }
    
    return $troubleshooting
}

# Troubleshoot application restrictions
$troubleshootingResult = Test-ApplicationRestrictions
$troubleshootingResult | Format-Table -AutoSize
```

#### System Restrictions Not Working
```powershell
# Function to troubleshoot system restrictions
function Test-SystemRestrictions {
    $troubleshooting = @()
    
    # Test 1: Check Command Prompt restriction
    $disableCMD = Get-ItemProperty -Path "HKCU:\Software\Policies\Microsoft\Windows\System" -Name "DisableCMD" -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "Command Prompt Restriction"
        Status = if ($disableCMD.DisableCMD -eq 1) { "Applied" } else { "Not Applied" }
        Details = "DisableCMD registry value: $($disableCMD.DisableCMD)"
    }
    
    # Test 2: Check Registry Editor restriction
    $disableRegEdit = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "DisableRegistryTools" -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "Registry Editor Restriction"
        Status = if ($disableRegEdit.DisableRegistryTools -eq 1) { "Applied" } else { "Not Applied" }
        Details = "DisableRegistryTools registry value: $($disableRegEdit.DisableRegistryTools)"
    }
    
    # Test 3: Check Task Manager restriction
    $disableTaskMgr = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\System" -Name "DisableTaskMgr" -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "Task Manager Restriction"
        Status = if ($disableTaskMgr.DisableTaskMgr -eq 1) { "Applied" } else { "Not Applied" }
        Details = "DisableTaskMgr registry value: $($disableTaskMgr.DisableTaskMgr)"
    }
    
    # Test 4: Check Control Panel restriction
    $disableControlPanel = Get-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoControlPanel" -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "Control Panel Restriction"
        Status = if ($disableControlPanel.NoControlPanel -eq 1) { "Applied" } else { "Not Applied" }
        Details = "NoControlPanel registry value: $($disableControlPanel.NoControlPanel)"
    }
    
    return $troubleshooting
}

# Troubleshoot system restrictions
$systemTroubleshooting = Test-SystemRestrictions
$systemTroubleshooting | Format-Table -AutoSize
```

## 📊 Endpoint Security Reporting

### Generate Security Compliance Report
```powershell
# Function to generate endpoint security compliance report
function New-EndpointSecurityComplianceReport {
    param(
        [string]$ReportPath = "C:\Reports\EndpointSecurityCompliance_$(Get-Date -Format 'yyyyMMdd_HHmmss').html"
    )
    
    # Get compliance data
    $complianceData = Test-EndpointSecurityCompliance
    
    # Generate HTML report
    $html = @"
<!DOCTYPE html>
<html>
<head>
    <title>Endpoint Security Compliance Report - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</title>
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
        .unknown { background-color: #fff3cd; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Endpoint Security Compliance Report</h1>
            <p>Generated on: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
        </div>
        
        <div class="summary">
            <h2>Compliance Summary</h2>
            <p>This report shows the current endpoint security compliance status.</p>
        </div>
        
        <div class="compliance-table">
            <h2>Security Policy Compliance</h2>
            <table class="compliance-table">
                <tr>
                    <th>Security Test</th>
                    <th>Status</th>
                    <th>Details</th>
                    <th>Compliance</th>
                </tr>
"@
    
    # Add compliance data
    foreach ($item in $complianceData) {
        $statusClass = switch ($item.Status) {
            "Enabled" { "compliant" }
            "Disabled" { "non-compliant" }
            default { "unknown" }
        }
        
        $compliance = if ($item.Status -eq "Enabled") { "Compliant" } else { "Non-Compliant" }
        
        $html += @"
                <tr class="$statusClass">
                    <td>$($item.Test)</td>
                    <td>$($item.Status)</td>
                    <td>$($item.Details)</td>
                    <td>$compliance</td>
                </tr>
"@
    }
    
    $html += @"
            </table>
        </div>
        
        <div class="compliance-table">
            <h2>Recommendations</h2>
            <table class="compliance-table">
                <tr>
                    <th>Area</th>
                    <th>Recommendation</th>
                    <th>Priority</th>
                </tr>
                <tr>
                    <td>Application Restrictions</td>
                    <td>Ensure all unauthorized applications are blocked</td>
                    <td>High</td>
                </tr>
                <tr>
                    <td>System Restrictions</td>
                    <td>Maintain strict system access controls</td>
                    <td>High</td>
                </tr>
                <tr>
                    <td>Windows Defender</td>
                    <td>Ensure real-time protection is always enabled</td>
                    <td>High</td>
                </tr>
                <tr>
                    <td>Windows Firewall</td>
                    <td>Maintain firewall rules and monitoring</td>
                    <td>Medium</td>
                </tr>
            </table>
        </div>
    </div>
</body>
</html>
"@
    
    # Save HTML report
    $html | Out-File -FilePath $ReportPath -Encoding UTF8
    
    Write-Host "Endpoint security compliance report generated: $ReportPath"
    return $ReportPath
}

# Generate compliance report
$complianceReport = New-EndpointSecurityComplianceReport
```

## 🖼️ Screenshots

### Application Restrictions
![Application Restrictions GPO](../screenshots/07-security/application-restrictions.png)

### System Restrictions
![System Restrictions](../screenshots/07-security/system-restrictions.png)

### Windows Defender Configuration
![Windows Defender](../screenshots/07-security/windows-defender.png)

### Windows Firewall Configuration
![Windows Firewall](../screenshots/07-security/windows-firewall.png)

## 📋 Endpoint Security Summary

| Security Area | Configuration | HR Department | Sales Department | IT Department | Status |
|----------------|---------------|----------------|------------------|----------------|---------|
| Application Restrictions | Disallowed List | ✅ Configured | ✅ Configured | ✅ Configured |
| System Restrictions | CMD/RegEdit/TaskMgr | ✅ Configured | ✅ Configured | ✅ Configured |
| Control Panel Restrictions | Disabled | ✅ Configured | ✅ Configured | ✅ Configured |
| Network Restrictions | Proxy/Disabled | ✅ Configured | ✅ Configured | ✅ Configured |
| Windows Defender | Real-time Protection | ✅ Configured | ✅ Configured | ✅ Configured |
| Windows Firewall | Enabled | ✅ Configured | ✅ Configured | ✅ Configured |

### Key Features Implemented
- ✅ Comprehensive application restrictions
- ✅ System access controls (CMD, RegEdit, Task Manager)
- ✅ Control Panel restrictions
- ✅ Network access limitations
- ✅ Windows Defender configuration
- ✅ Windows Firewall configuration
- ✅ Security compliance monitoring
- ✅ Automated reporting and troubleshooting

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
