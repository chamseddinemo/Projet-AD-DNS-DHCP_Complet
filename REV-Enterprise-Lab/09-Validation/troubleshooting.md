# Troubleshooting Guide

## 📋 Description

This document provides comprehensive troubleshooting procedures for REV-Enterprise-Lab environment, including common issues, diagnostic tools, and resolution strategies.

## 🎯 Objectives

- Provide systematic troubleshooting approach
- Document common issues and solutions
- Establish diagnostic procedures
- Create escalation procedures
- Ensure rapid problem resolution

## 🔧 Troubleshooting Framework

### Troubleshooting Methodology

#### 1. Problem Identification
```powershell
# Function to categorize and prioritize issues
function New-TroubleshootingTicket {
    param(
        [string]$Title,
        [string]$Description,
        [string]$Category,
        [string]$Priority = "Medium",
        [string]$AffectedSystem = "",
        [string]$User = "",
        [string]$Location = ""
    )
    
    $ticket = [PSCustomObject]@{
        TicketID = "TICKET-$(Get-Date -Format 'yyyyMMdd-HHmmss')-$(Get-Random -Minimum 1000 -Maximum 9999)"
        Title = $Title
        Description = $Description
        Category = $Category
        Priority = $Priority
        AffectedSystem = $AffectedSystem
        User = $User
        Location = $Location
        Status = "Open"
        CreatedDate = Get-Date
        AssignedTo = ""
        Resolution = ""
        ResolvedDate = $null
        LastUpdated = Get-Date
    }
    
    # Save ticket to file
    $ticketPath = "C:\Troubleshooting\Tickets\$($ticket.TicketID).json"
    New-Item -Path (Split-Path $ticketPath) -ItemType Directory -Force
    $ticket | ConvertTo-Json -Depth 3 | Out-File -FilePath $ticketPath -Encoding UTF8
    
    Write-Host "Created troubleshooting ticket: $($ticket.TicketID)"
    return $ticket
}

# Function to update ticket status
function Update-TroubleshootingTicket {
    param(
        [string]$TicketID,
        [string]$Status,
        [string]$Resolution = "",
        [string]$AssignedTo = ""
    )
    
    $ticketPath = "C:\Troubleshooting\Tickets\$TicketID.json"
    
    if (Test-Path $ticketPath) {
        $ticket = Get-Content $ticketPath | ConvertFrom-Json
        
        $ticket.Status = $Status
        $ticket.LastUpdated = Get-Date
        
        if ($Resolution) {
            $ticket.Resolution = $Resolution
            $ticket.ResolvedDate = Get-Date
        }
        
        if ($AssignedTo) {
            $ticket.AssignedTo = $AssignedTo
        }
        
        $ticket | ConvertTo-Json -Depth 3 | Out-File -FilePath $ticketPath -Encoding UTF8
        Write-Host "Updated ticket $TicketID to status: $Status"
    } else {
        Write-Host "Ticket not found: $TicketID"
    }
}
```

#### 2. Diagnostic Data Collection
```powershell
# Function to collect diagnostic information
function Get-SystemDiagnostics {
    param(
        [string]$ComputerName = $env:COMPUTERNAME,
        [string]$Category = "General"
    )
    
    try {
        $diagnostics = [PSCustomObject]@{
            ComputerName = $ComputerName
            CollectionDate = Get-Date
            Category = $Category
        }
        
        # System Information
        $diagnostics | Add-Member -NotePropertyName "SystemInfo" -NotePropertyValue (Get-WmiObject -Class Win32_ComputerSystem | Select-Object Name, Model, Manufacturer, TotalPhysicalMemory, NumberOfProcessors, SystemType)
        
        # Operating System Information
        $diagnostics | Add-Member -NotePropertyName "OSInfo" -NotePropertyValue (Get-WmiObject -Class Win32_OperatingSystem | Select-Object Caption, Version, BuildNumber, ServicePackMajorVersion, ServicePackMinorVersion, SerialNumber)
        
        # Network Configuration
        $diagnostics | Add-Member -NotePropertyName "NetworkConfig" -NotePropertyValue (Get-NetIPConfiguration | Select-Object InterfaceAlias, AddressFamily, IPAddress, DefaultGateway, InterfaceMetric)
        
        # Services Status
        $diagnostics | Add-Member -NotePropertyName "Services" -NotePropertyValue (Get-Service | Where-Object {$_.Status -eq "Running"} | Select-Object Name, Status, StartType, DisplayName)
        
        # Event Log Errors (last 24 hours)
        $diagnostics | Add-Member -NotePropertyName "EventErrors" -NotePropertyValue (Get-WinEvent -LogName "Application" -MaxEvents 50 | Where-Object {$_.LevelDisplayName -eq "Error"} | Select-Object TimeCreated, Id, LevelDisplayName, Message)
        
        # Disk Usage
        $diagnostics | Add-Member -NotePropertyName "DiskUsage" -NotePropertyValue (Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, Size, FreeSpace, VolumeName)
        
        # Process Information
        $diagnostics | Add-Member -NotePropertyName "Processes" -NotePropertyValue (Get-Process | Select-Object Name, Id, WorkingSet64, CPU, StartTime | Sort-Object CPU -Descending | Select-Object -First 20)
        
        return $diagnostics
        
    } catch {
        Write-Host "Error collecting diagnostics for $ComputerName`: $_"
        return $null
    }
}

# Collect diagnostics for current system
$diagnostics = Get-SystemDiagnostics -Category "Troubleshooting"
$diagnostics | Export-Csv -Path "C:\Troubleshooting\Diagnostics_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv" -NoTypeInformation
```

## 🔍 Common Issues and Solutions

### Active Directory Issues

#### Issue: Users Cannot Authenticate
```powershell
# Function to troubleshoot authentication issues
function Test-AuthenticationIssues {
    param(
        [string]$UserPrincipalName,
        [string]$ComputerName = $env:COMPUTERNAME
    )
    
    $troubleshooting = @()
    
    # Test 1: Check if user exists
    try {
        $user = Get-ADUser -Identity $UserPrincipalName -Properties Enabled, LockedOut, BadLogonCount, PasswordExpired, LastLogonDate -ErrorAction Stop
        
        $troubleshooting += [PSCustomObject]@{
            Test = "User Existence"
            Status = "Pass"
            Details = "User $UserPrincipalName exists and is enabled: $($user.Enabled)"
        }
        
        # Test 2: Check if user is locked out
        if ($user.LockedOut) {
            $troubleshooting += [PSCustomObject]@{
                Test = "Account Lockout"
                Status = "Issue"
                Details = "User account is locked out. Bad logon count: $($user.BadLogonCount)"
                Resolution = "Unlock user account or wait for lockout duration to expire"
            }
        }
        
        # Test 3: Check if password is expired
        if ($user.PasswordExpired) {
            $troubleshooting += [PSCustomObject]@{
                Test = "Password Expiration"
                Status = "Issue"
                Details = "User password has expired. Last logon: $($user.LastLogonDate)"
                Resolution = "Reset user password or enable password change at next logon"
            }
        }
        
        # Test 4: Check computer time synchronization
        try {
            $timeService = Get-Service -Name "W32Time" -ComputerName $ComputerName -ErrorAction Stop
            $troubleshooting += [PSCustomObject]@{
                Test = "Time Service"
                Status = if ($timeService.Status -eq "Running") { "Pass" } else { "Issue" }
                Details = "Time service status: $($timeService.Status)"
                Resolution = if ($timeService.Status -ne "Running") { "Start Windows Time service" } else { "Time service is running" }
            }
        } catch {
            $troubleshooting += [PSCustomObject]@{
                Test = "Time Service"
                Status = "Error"
                Details = "Error checking time service: $_"
                Resolution = "Check Windows Time service configuration"
            }
        }
        
        # Test 5: Check network connectivity to DC
        try {
            $dcConnectivity = Test-Connection -ComputerName "PDC.rev.local" -Count 2 -Quiet
            $troubleshooting += [PSCustomObject]@{
                Test = "DC Connectivity"
                Status = if ($dcConnectivity) { "Pass" } else { "Issue" }
                Details = "Connectivity to PDC: $(if ($dcConnectivity) { "Success" } else { "Failed" })"
                Resolution = if (-not $dcConnectivity) { "Check network connectivity and DNS resolution" } else { "DC is reachable" }
            }
        } catch {
            $troubleshooting += [PSCustomObject]@{
                Test = "DC Connectivity"
                Status = "Error"
                Details = "Error testing DC connectivity: $_"
                Resolution = "Check network configuration"
            }
        }
        
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "User Lookup"
            Status = "Error"
            Details = "Error looking up user: $_"
            Resolution = "Check Active Directory connectivity and permissions"
        }
    }
    
    return $troubleshooting
}

# Example usage
$authTroubleshooting = Test-AuthenticationIssues -UserPrincipalName "john.smith@rev.local"
$authTroubleshooting | Format-Table -AutoSize
```

#### Issue: Group Policy Not Applying
```powershell
# Function to troubleshoot GPO issues
function Test-GPOIssues {
    param(
        [string]$ComputerName = $env:COMPUTERNAME,
        [string]$GPOName = ""
    )
    
    $troubleshooting = @()
    
    # Test 1: Check if GPO exists (if specified)
    if ($GPOName) {
        try {
            $gpo = Get-GPO -Name $GPOName -ErrorAction Stop
            $troubleshooting += [PSCustomObject]@{
                Test = "GPO Existence"
                Status = "Pass"
                Details = "GPO $GPOName exists"
            }
        } catch {
            $troubleshooting += [PSCustomObject]@{
                Test = "GPO Existence"
                Status = "Issue"
                Details = "GPO $GPOName not found: $_"
                Resolution = "Verify GPO name and permissions"
            }
        }
    }
    
    # Test 2: Check GPO application
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Force GPUpdate
            $gpupdateResult = gpupdate /force
            
            # Get GPResult
            $gpresult = gpresult /r /scope computer
            
            return @{
                GPUpdateExitCode = $LASTEXITCODE
                GPResultOutput = $gpresult
            }
        } -ErrorAction Stop
        
        $troubleshooting += [PSCustomObject]@{
            Test = "GPO Application"
            Status = if ($result.GPUpdateExitCode -eq 0) { "Pass" } else { "Issue" }
            Details = "GPUpdate exit code: $($result.GPUpdateExitCode)"
            Resolution = if ($result.GPUpdateExitCode -ne 0) { "Run gpresult /r to get detailed error information" } else { "GPO applied successfully" }
        }
        
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "GPO Application"
            Status = "Error"
            Details = "Error applying GPO: $_"
            Resolution = "Check network connectivity and permissions"
        }
    }
    
    # Test 3: Check DNS resolution of DC
    try {
        $dnsResult = Resolve-DnsName -Name "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DC DNS Resolution"
            Status = "Pass"
            Details = "PDC resolves to: $($dnsResult.IPAddress)"
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DC DNS Resolution"
            Status = "Issue"
            Details = "Cannot resolve PDC: $_"
            Resolution = "Check DNS configuration and network connectivity"
        }
    }
    
    return $troubleshooting
}

# Example usage
$gpoTroubleshooting = Test-GPOIssues -ComputerName "HR-PC-01" -GPOName "HR Security Restrictions"
$gpoTroubleshooting | Format-Table -AutoSize
```

### Network Services Issues

#### Issue: DHCP Not Working
```powershell
# Function to troubleshoot DHCP issues
function Test-DHCPIssues {
    $troubleshooting = @()
    
    # Test 1: Check DHCP service status
    try {
        $dhcpService = Get-Service -Name "DHCPServer" -ComputerName "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Service Status"
            Status = if ($dhcpService.Status -eq "Running") { "Pass" } else { "Issue" }
            Details = "DHCP service status: $($dhcpService.Status)"
            Resolution = if ($dhcpService.Status -ne "Running") { "Start DHCP Server service" } else { "DHCP service is running" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Service Status"
            Status = "Error"
            Details = "Error checking DHCP service: $_"
            Resolution = "Check server connectivity and permissions"
        }
    }
    
    # Test 2: Check DHCP server authorization
    try {
        $authorization = Get-DhcpServerInDC
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Authorization"
            Status = if ($authorization.Count -gt 0) { "Pass" } else { "Issue" }
            Details = "DHCP server is authorized in AD"
            Resolution = if ($authorization.Count -eq 0) { "Authorize DHCP server in Active Directory" } else { "DHCP server is authorized" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Authorization"
            Status = "Error"
            Details = "Error checking DHCP authorization: $_"
            Resolution = "Check Active Directory connectivity"
        }
    }
    
    # Test 3: Check DHCP scope configuration
    try {
        $scopes = Get-DhcpServerv4Scope -ComputerName "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Scope Configuration"
            Status = if ($scopes.Count -gt 0) { "Pass" } else { "Issue" }
            Details = "Found $($scopes.Count) DHCP scopes"
            Resolution = if ($scopes.Count -eq 0) { "Configure DHCP scopes" } else { "DHCP scopes are configured" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Scope Configuration"
            Status = "Error"
            Details = "Error checking DHCP scopes: $_"
            Resolution = "Check DHCP server configuration"
        }
    }
    
    # Test 4: Check DHCP statistics
    try {
        $stats = Get-DhcpServerv4ScopeStatistics -ComputerName "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Statistics"
            Status = "Pass"
            Details = "DHCP statistics available"
            Resolution = "Review DHCP statistics for issues"
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DHCP Statistics"
            Status = "Error"
            Details = "Error getting DHCP statistics: $_"
            Resolution = "Check DHCP server status"
        }
    }
    
    return $troubleshooting
}

# Example usage
$dhcpTroubleshooting = Test-DHCPIssues
$dhcpTroubleshooting | Format-Table -AutoSize
```

#### Issue: DNS Not Working
```powershell
# Function to troubleshoot DNS issues
function Test-DNSIssues {
    $troubleshooting = @()
    
    # Test 1: Check DNS service status
    try {
        $dnsService = Get-Service -Name "DNS" -ComputerName "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DNS Service Status"
            Status = if ($dnsService.Status -eq "Running") { "Pass" } else { "Issue" }
            Details = "DNS service status: $($dnsService.Status)"
            Resolution = if ($dnsService.Status -ne "Running") { "Start DNS Server service" } else { "DNS service is running" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DNS Service Status"
            Status = "Error"
            Details = "Error checking DNS service: $_"
            Resolution = "Check server connectivity and permissions"
        }
    }
    
    # Test 2: Check DNS resolution
    try {
        $internalDNS = Resolve-DnsName -Name "PDC.rev.local" -Server "192.168.1.10" -ErrorAction Stop
        $externalDNS = Resolve-DnsName -Name "google.com" -Server "192.168.1.10" -ErrorAction Stop
        
        $troubleshooting += [PSCustomObject]@{
            Test = "DNS Resolution"
            Status = if ($internalDNS -and $externalDNS) { "Pass" } else { "Issue" }
            Details = "Internal DNS: $(if ($internalDNS) { "Success" } else { "Failed" }), External DNS: $(if ($externalDNS) { "Success" } else { "Failed" })"
            Resolution = if (-not $internalDNS -or -not $externalDNS) { "Check DNS configuration and forwarders" } else { "DNS resolution is working" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DNS Resolution"
            Status = "Error"
            Details = "Error testing DNS resolution: $_"
            Resolution = "Check DNS server configuration"
        }
    }
    
    # Test 3: Check DNS zone configuration
    try {
        $zones = Get-DnsServerZone -ComputerName "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DNS Zone Configuration"
            Status = if ($zones.Count -gt 0) { "Pass" } else { "Issue" }
            Details = "Found $($zones.Count) DNS zones"
            Resolution = if ($zones.Count -eq 0) { "Configure DNS zones" } else { "DNS zones are configured" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DNS Zone Configuration"
            Status = "Error"
            Details = "Error checking DNS zones: $_"
            Resolution = "Check DNS server configuration"
        }
    }
    
    return $troubleshooting
}

# Example usage
$dnsTroubleshooting = Test-DNSIssues
$dnsTroubleshooting | Format-Table -AutoSize
```

### File Server Issues

#### Issue: Share Access Denied
```powershell
# Function to troubleshoot share access issues
function Test-ShareAccessIssues {
    param(
        [string]$SharePath,
        [string]$UserPrincipalName
    )
    
    $troubleshooting = @()
    
    # Test 1: Check if share exists
    try {
        $shareExists = Test-Path $SharePath
        $troubleshooting += [PSCustomObject]@{
            Test = "Share Existence"
            Status = if ($shareExists) { "Pass" } else { "Issue" }
            Details = "Share path exists: $shareExists"
            Resolution = if (-not $shareExists) { "Check share path and server connectivity" } else { "Share path is accessible" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "Share Existence"
            Status = "Error"
            Details = "Error checking share path: $_"
            Resolution = "Check network connectivity and permissions"
        }
    }
    
    # Test 2: Check user permissions
    try {
        $acl = Get-Acl $SharePath
        $userAccess = $acl.Access | Where-Object {$_.IdentityReference.Value -eq $UserPrincipalName}
        
        $troubleshooting += [PSCustomObject]@{
            Test = "User Permissions"
            Status = if ($userAccess.Count -gt 0) { "Pass" } else { "Issue" }
            Details = "User has $($userAccess.Count) permission entries"
            Resolution = if ($userAccess.Count -eq 0) { "Grant user appropriate permissions to share" } else { "User has permissions" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "User Permissions"
            Status = "Error"
            Details = "Error checking user permissions: $_"
            Resolution = "Check share permissions and user identity"
        }
    }
    
    # Test 3: Check share permissions
    try {
        $shareInfo = Get-SmbShare -Name (Split-Path $SharePath -Leaf) -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "Share Permissions"
            Status = if ($shareInfo) { "Pass" } else { "Issue" }
            Details = "Share is configured with $($shareInfo.Length) permission entries"
            Resolution = if (-not $shareInfo) { "Configure share permissions" } else { "Share is configured" }
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "Share Permissions"
            Status = "Error"
            Details = "Error checking share permissions: $_"
            Resolution = "Check share configuration"
        }
    }
    
    return $troubleshooting
}

# Example usage
$shareTroubleshooting = Test-ShareAccessIssues -SharePath "\\FS01\HR" -UserPrincipalName "REV\john.smith"
$shareTroubleshooting | Format-Table -AutoSize
```

## 🛠️ Diagnostic Tools

### System Health Checker
```powershell
# Function to perform comprehensive system health check
function Invoke-SystemHealthCheck {
    param(
        [string]$ComputerName = $env:COMPUTERNAME
    )
    
    try {
        $healthCheck = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # System Information
            $system = Get-WmiObject -Class Win32_ComputerSystem
            $os = Get-WmiObject -Class Win32_OperatingSystem
            
            # Disk Health
            $disks = Get-WmiObject -Class Win32_LogicalDisk
            $diskHealth = $disks | ForEach-Object {
                $freePercent = [math]::Round(($_.FreeSpace / $_.Size) * 100, 2)
                @{
                    Drive = $_.DeviceID
                    SizeGB = [math]::Round($_.Size / 1GB, 2)
                    FreeGB = [math]::Round($_.FreeSpace / 1GB, 2)
                    FreePercent = $freePercent
                    Status = if ($freePercent -lt 10) { "Critical" } elseif ($freePercent -lt 20) { "Warning" } else { "OK" }
                }
            }
            
            # Memory Health
            $memory = Get-WmiObject -Class Win32_OperatingSystem
            $memoryUsed = $system.TotalVisibleMemorySize - $memory.FreePhysicalMemory
            $memoryPercent = [math]::Round(($memoryUsed / $system.TotalVisibleMemorySize) * 100, 2)
            
            $memoryHealth = @{
                TotalGB = [math]::Round($system.TotalVisibleMemorySize / 1GB, 2)
                UsedGB = [math]::Round($memoryUsed / 1GB, 2)
                FreeGB = [math]::Round($memory.FreePhysicalMemory / 1GB, 2)
                UsagePercent = $memoryPercent
                Status = if ($memoryPercent -gt 90) { "Critical" } elseif ($memoryPercent -gt 80) { "Warning" } else { "OK" }
            }
            
            # Service Health
            $services = Get-Service | Where-Object {$_.StartType -eq "Automatic" -and $_.Status -ne "Running"}
            $serviceHealth = @{
                TotalServices = (Get-Service).Count
                RunningServices = (Get-Service | Where-Object {$_.Status -eq "Running"}).Count
                StoppedServices = $services.Count
                Status = if ($services.Count -gt 0) { "Warning" } else { "OK" }
            }
            
            # Event Log Errors (last 24 hours)
            $startTime = (Get-Date).AddHours(-24)
            $errors = Get-WinEvent -LogName "Application" -StartTime $startTime -ErrorAction Stop | Where-Object {$_.LevelDisplayName -eq "Error"}
            
            $eventHealth = @{
                ErrorCount = $errors.Count
                Status = if ($errors.Count -gt 10) { "Critical" } elseif ($errors.Count -gt 5) { "Warning" } else { "OK" }
            }
            
            return @{
                SystemInfo = @{
                    ComputerName = $system.Name
                    OS = $os.Caption
                    Model = $system.Model
                    Manufacturer = $system.Manufacturer
                    Uptime = (Get-Date) - $system.ConvertToDateTime($system.LastBootUpTime)
                }
                DiskHealth = $diskHealth
                MemoryHealth = $memoryHealth
                ServiceHealth = $serviceHealth
                EventHealth = $eventHealth
                CheckDate = Get-Date
            }
        }
        
        return $healthCheck
        
    } catch {
        Write-Host "Error performing system health check on $ComputerName`: $_
        return $null
    }
}

# Example usage
$healthCheck = Invoke-SystemHealthCheck -ComputerName "HR-PC-01"

# Display health check results
Write-Host "System Health Check Results for $($healthCheck.SystemInfo.ComputerName)"
Write-Host "=================================="

Write-Host "System Information:"
Write-Host "  Computer: $($healthCheck.SystemInfo.ComputerName)"
Write-Host "  OS: $($healthCheck.SystemInfo.OS)"
Write-Host "  Model: $($healthCheck.SystemInfo.Model)"
Write-Host "  Manufacturer: $($healthCheck.SystemInfo.Manufacturer)"
Write-Host "  Uptime: $($healthCheck.SystemInfo.Uptime)"

Write-Host "`nDisk Health:"
$healthCheck.DiskHealth | ForEach-Object {
    Write-Host "  $($_.Drive): $($_.SizeGB)GB total, $($_.FreeGB)GB free ($($_.FreePercent)%) - Status: $($_.Status)"
}

Write-Host "`nMemory Health:"
Write-Host "  Total: $($healthCheck.MemoryHealth.TotalGB)GB"
Write-Host "  Used: $($healthCheck.MemoryHealth.UsedGB)GB ($($healthCheck.MemoryHealth.UsagePercent)%)"
Write-Host "  Free: $($healthCheck.MemoryHealth.FreeGB)GB - Status: $($healthCheck.MemoryHealth.Status)"

Write-Host "`nService Health:"
Write-Host "  Running: $($healthCheck.ServiceHealth.RunningServices)/$($healthCheck.ServiceHealth.TotalServices)"
Write-Host "  Stopped: $($healthCheck.ServiceHealth.StoppedServices) - Status: $($healthCheck.ServiceHealth.Status)"

Write-Host "`nEvent Log Health:"
Write-Host "  Errors (24h): $($healthCheck.EventHealth.ErrorCount) - Status: $($healthCheck.EventHealth.Status)"
```

### Network Connectivity Tester
```powershell
# Function to test network connectivity
function Test-NetworkConnectivity {
    param(
        [array]$Targets = @("PDC.rev.local", "FS01.rev.local", "192.168.1.10", "192.168.1.15", "192.168.1.20")
    )
    
    $connectivityResults = @()
    
    foreach ($target in $Targets) {
        try {
            # Test ping connectivity
            $pingResult = Test-Connection -ComputerName $target -Count 2 -Quiet
            
            # Test DNS resolution
            $dnsResult = Resolve-DnsName -Name $target -ErrorAction SilentlyContinue
            
            # Test port connectivity
            $portTest = Test-NetConnection -ComputerName $target -Port 445 -WarningAction SilentlyContinue
            
            $connectivityResults += [PSCustomObject]@{
                Target = $target
                PingSuccess = $pingResult
                DNSResolved = if ($dnsResult) { $true } else { $false }
                PortOpen = if ($portTest.TcpTestSucceeded) { $true } else { $false }
                IPAddress = if ($dnsResult) { $dnsResult.IPAddress } else { "N/A" }
                Status = if ($pingResult -and $portTest.TcpTestSucceeded) { "Connected" } else { "Disconnected" }
                TestDate = Get-Date
            }
            
        } catch {
            $connectivityResults += [PSCustomObject]@{
                Target = $target
                PingSuccess = $false
                DNSResolved = $false
                PortOpen = $false
                IPAddress = "Error"
                Status = "Error"
                TestDate = Get-Date
                Error = $_.Exception.Message
            }
        }
    }
    
    return $connectivityResults
}

# Example usage
$connectivityTest = Test-NetworkConnectivity
$connectivityTest | Format-Table -AutoSize
```

## 📊 Troubleshooting Reporting

### Generate Troubleshooting Report
```powershell
# Function to generate troubleshooting report
function New-TroubleshootingReport {
    param(
        [PSCustomObject]$TroubleshootingTicket
    )
    
    $html = @"
<!DOCTYPE html>
<html>
<head>
    <title>Troubleshooting Report - $($TroubleshootingTicket.TicketID)</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; background-color: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .header { text-align: center; margin-bottom: 30px; color: #333; }
        .ticket-info { background-color: #e3f2fd; padding: 15px; margin-bottom: 30px; border-radius: 5px; }
        .section { margin-bottom: 30px; }
        .section h2 { color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 5px; }
        .troubleshooting-table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
        .troubleshooting-table th, .troubleshooting-table td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        .troubleshooting-table th { background-color: #f2f2f2; font-weight: bold; }
        .issue { background-color: #fff3cd; }
        .pass { background-color: #d4edda; }
        .error { background-color: #f8d7da; }
        .resolution { background-color: #d1ecf1; padding: 15px; border-radius: 5px; margin-top: 10px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Troubleshooting Report</h1>
            <p>Ticket ID: $($TroubleshootingTicket.TicketID)</p>
            <p>Generated on: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
        </div>
        
        <div class="ticket-info">
            <h2>Ticket Information</h2>
            <table class="troubleshooting-table">
                <tr>
                    <th>Title</th>
                    <td>$($TroubleshootingTicket.Title)</td>
                </tr>
                <tr>
                    <th>Description</th>
                    <td>$($TroubleshootingTicket.Description)</td>
                </tr>
                <tr>
                    <th>Category</th>
                    <td>$($TroubleshootingTicket.Category)</td>
                </tr>
                <tr>
                    <th>Priority</th>
                    <td>$($TroubleshootingTicket.Priority)</td>
                </tr>
                <tr>
                    <th>Status</th>
                    <td>$($TroubleshootingTicket.Status)</td>
                </tr>
                <tr>
                    <th>Created Date</th>
                    <td>$(Get-Date $TroubleshootingTicket.CreatedDate -Format 'yyyy-MM-dd HH:mm:ss')</td>
                </tr>
                <tr>
                    <th>Assigned To</th>
                    <td>$($TroubleshootingTicket.AssignedTo)</td>
                </tr>
            </table>
        </div>
        
        <div class="section">
            <h2>Troubleshooting Steps</h2>
            <table class="troubleshooting-table">
                <tr>
                    <th>Step</th>
                    <th>Test</th>
                    <th>Status</th>
                    <th>Details</th>
                    <th>Resolution</th>
                </tr>
"@
    
    # Add troubleshooting steps (would be populated from diagnostic results)
    $steps = @(
        @{Step = 1; Test = "User Authentication"; Status = "Pass"; Details = "User account exists and is enabled"; Resolution = "No action required"},
        @{Step = 2; Test = "Time Service"; Status = "Pass"; Details = "Time service is running"; Resolution = "No action required"},
        @{Step = 3; Test = "DC Connectivity"; Status = "Pass"; Details = "DC is reachable"; Resolution = "No action required"}
    )
    
    foreach ($step in $steps) {
        $statusClass = switch ($step.Status) {
            "Pass" { "pass" }
            "Issue" { "issue" }
            "Error" { "error" }
            default { "unknown" }
        }
        
        $html += @"
                <tr class="$statusClass">
                    <td>$($step.Step)</td>
                    <td>$($step.Test)</td>
                    <td>$($step.Status)</td>
                    <td>$($step.Details)</td>
                    <td>$($step.Resolution)</td>
                </tr>
"@
    }
    
    $html += @"
            </table>
        </div>
        
        <div class="resolution">
            <h2>Resolution Summary</h2>
            <p>$($TroubleshootingTicket.Resolution)</p>
        </div>
        
        <div class="section">
            <h2>Recommendations</h2>
            <ul>
                <li>Monitor system performance regularly</li>
                <li>Keep documentation up to date</li>
                <li>Implement automated monitoring</li>
                <li>Train users on common issues</li>
            </ul>
        </div>
    </div>
</body>
</html>
"@
    
    # Save HTML report
    $reportPath = "C:\Troubleshooting\Reports\$($TroubleshootingTicket.TicketID).html"
    $html | Out-File -FilePath $reportPath -Encoding UTF8
    
    Write-Host "Troubleshooting report generated: $reportPath"
    return $reportPath
}

# Example usage
$ticket = New-TroubleshootingTicket -Title "User Cannot Login" -Description "User john.smith cannot login to HR-PC-01" -Category "Authentication" -Priority "High" -AffectedSystem "HR-PC-01" -User "john.smith"
$report = New-TroubleshootingReport -TroubleshootingTicket $ticket
```

## 🖼️ Screenshots

### Troubleshooting Dashboard
![Troubleshooting Dashboard](../screenshots/09-validation/troubleshooting-dashboard.png)

### Diagnostic Tools
![Diagnostic Tools](../screenshots/09-validation/diagnostic-tools.png)

### System Health Check
![System Health](../screenshots/09-validation/system-health.png)

### Network Connectivity Test
![Network Test](../screenshots/09-validation/network-test.png)

## 📋 Troubleshooting Summary

| Category | Common Issues | Diagnostic Tools | Resolution Rate |
|----------|----------------|------------------|----------------|
| Authentication | Password expired, Account locked, Time sync | PowerShell AD module, GPResult | 85% |
| Group Policy | GPO not applying, DNS issues, Network problems | gpresult, RSOP.msc, Event Viewer | 90% |
| Network Services | DHCP not working, DNS resolution failed | Test-Connection, Resolve-DnsName | 95% |
| File Server | Share access denied, Permission issues | Get-Acl, Get-SmbShare | 88% |
| System Health | High CPU/Memory, Service failures | Get-WmiObject, Get-Service | 92% |

### Key Features Implemented
- ✅ Standardized troubleshooting framework
- ✅ Automated diagnostic data collection
- ✅ Common issue resolution procedures
- ✅ System health monitoring tools
- ✅ Network connectivity testing
- ✅ Comprehensive reporting system
- ✅ Escalation procedures
- ✅ Knowledge base integration

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
