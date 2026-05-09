# DHCP Server Configuration

## 📋 Description

This document provides comprehensive procedures for installing and configuring DHCP (Dynamic Host Configuration Protocol) services in the REV-Enterprise-Lab environment, including scope configuration, reservations, and management procedures.

## 🎯 Objectives

- Install and configure DHCP Server role
- Create and configure DHCP scopes for different network segments
- Implement DHCP reservations for critical devices
- Configure DHCP options and settings
- Establish DHCP redundancy and failover procedures

## 🛠️ DHCP Server Installation

### Prerequisites
- **Server OS**: Windows Server 2019/2022
- **Static IP**: 192.168.1.10 (PDC)
- **Administrative Rights**: Domain Administrator or equivalent
- **Network Configuration**: Proper IP addressing and connectivity

### DHCP Server Role Installation

#### GUI Installation
1. Open **Server Manager**
2. Click **Add roles and features**
3. Select **Role-based or feature-based installation**
4. Choose the target server (PDC)
5. Select **DHCP Server** role
6. Add required features automatically
7. Click **Install** and wait for completion
8. Complete DHCP post-installation configuration

#### PowerShell Installation
```powershell
# Install DHCP Server role
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Verify installation
Get-WindowsFeature -Name DHCP

# Restart DHCP service if needed
Restart-Service -Name DHCPServer -Force
```

### DHCP Server Authorization
```powershell
# Authorize DHCP server in Active Directory
Add-DhcpServerInDC -DnsName "PDC.rev.local" -IPAddress 192.168.1.10

# Verify authorization
Get-DhcpServerInDC
```

## 📊 DHCP Scope Configuration

### Primary Client Scope (192.168.1.64/26)

#### Scope Configuration
```powershell
# Create primary client scope
Add-DhcpServerv4Scope -ComputerName "PDC.rev.local" -Name "Client Network" -Description "Primary scope for client computers" -StartRange 192.168.1.66 -EndRange 192.168.1.126 -SubnetMask 255.255.255.192 -State Active

# Configure scope options
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -DnsServer 192.168.1.10 -Router 192.168.1.65 -DomainName "rev.local" -DnsDomain "rev.local"

# Set lease duration
Set-DhcpServerv4Scope -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -LeaseDuration 8.00:00:00

# Configure DNS dynamic updates
Set-DhcpServerv4DnsSetting -ComputerName "PDC.rev.local" -DynamicUpdates Always -DeleteDnsRRonLeaseExpiry $true -UpdateDnsRRForOlderClients $true
```

#### Scope Options Configuration
```powershell
# Configure standard scope options
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -OptionId 3 -Value 192.168.1.65  # Router
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -OptionId 6 -Value 192.168.1.10  # DNS Server
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -OptionId 15 -Value "rev.local"  # Domain Name
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -OptionId 44 -Value 192.168.1.10  # WINS/NBNS Server
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -OptionId 46 -Value 0x08  # Node Type (H-node)
```

### Guest Network Scope (192.168.1.128/25)

#### Guest Scope Configuration
```powershell
# Create guest network scope
Add-DhcpServerv4Scope -ComputerName "PDC.rev.local" -Name "Guest Network" -Description "Scope for guest and temporary devices" -StartRange 192.168.1.130 -EndRange 192.168.1.239 -SubnetMask 255.255.255.128 -State Active

# Configure guest scope options
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.128 -DnsServer 8.8.8.8,8.8.4.4 -Router 192.168.1.129 -LeaseDuration 2.00:00:00

# Set shorter lease duration for guest devices
Set-DhcpServerv4Scope -ComputerName "PDC.rev.local" -ScopeId 192.168.1.128 -LeaseDuration 2.00:00:00

# Configure different DNS settings for guests
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -ScopeId 192.168.1.128 -OptionId 6 -Value 8.8.8.8,8.8.4.4  # Public DNS
```

## 🔧 DHCP Exclusions Configuration

### Network Device Exclusions
```powershell
# Exclude range for network devices (Printers, APs, etc.)
Add-DhcpServerv4ExclusionRange -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -StartRange 192.168.1.80 -EndRange 192.168.1.85

# Exclude range for servers (should be static)
Add-DhcpServerv4ExclusionRange -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -StartRange 192.168.1.10 -EndRange 192.168.1.20

# Exclude range for management devices
Add-DhcpServerv4ExclusionRange -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -StartRange 192.168.1.240 -EndRange 192.168.1.254
```

### Exclusion Summary
| Range | Purpose | Status |
|-------|---------|--------|
| 192.168.1.80-85 | Network Devices (Printers, APs) | ✅ Configured |
| 192.168.1.10-20 | Server Infrastructure | ✅ Configured |
| 192.168.1.240-254 | Management Devices | ✅ Configured |

## 🖥️ DHCP Reservations

### Critical Device Reservations

#### HR Workstation Reservation
```powershell
# Reserve IP for HRPC01
Add-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -IPAddress 192.168.1.200 -ClientId "00-15-5D-01-02-01" -Name "HRPC01" -Description "HR Manager Workstation"

# Verify reservation
Get-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 | Where-Object {$_.IPAddress -eq "192.168.1.200"}
```

#### Server Reservations
```powershell
# Reserve IP for File Server (if needed)
Add-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -IPAddress 192.168.1.15 -ClientId "00-15-5D-01-01-15" -Name "FS01" -Description "File Server"

# Reserve IP for Application Server (if needed)
Add-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -IPAddress 192.168.1.20 -ClientId "00-15-5D-01-01-20" -Name "APP01" -Description "Application Server"
```

#### Network Device Reservations
```powershell
# Reserve IP for Network Printer
Add-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -IPAddress 192.168.1.81 -ClientId "00-11-22-33-44-55" -Name "HR-Printer-01" -Description "HR Department Printer"

# Reserve IP for Wireless Access Point
Add-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId 192.168.1.64 -IPAddress 192.168.1.82 -ClientId "00-11-22-33-44-66" -Name "WAP-01" -Description "Main Office Wireless AP"
```

### Reservation Management
```powershell
# Function to create DHCP reservation
function New-DhcpReservation {
    param(
        [string]$ComputerName,
        [string]$IPAddress,
        [string]$MACAddress,
        [string]$Description,
        [string]$ScopeId = "192.168.1.64"
    )
    
    try {
        Add-DhcpServerv4Reservation -ComputerName "PDC.rev.local" -ScopeId $ScopeId -IPAddress $IPAddress -ClientId $MACAddress -Name $ComputerName -Description $Description
        Write-Host "Created reservation for $ComputerName`: $IPAddress"
    } catch {
        Write-Host "Error creating reservation for $ComputerName`: $_"
    }
}

# Example usage
New-DhcpReservation -ComputerName "TEST-PC-01" -IPAddress "192.168.1.201" -MACAddress "00-15-5D-01-02-02" -Description "Test Workstation"
```

## 📊 DHCP Server Management

### DHCP Statistics and Monitoring

#### Scope Statistics
```powershell
# Get scope statistics
$scopeStats = Get-DhcpServerv4ScopeStatistics -ComputerName "PDC.rev.local"

foreach ($stat in $scopeStats) {
    Write-Host "Scope: $($stat.ScopeId)"
    Write-Host "  Total Addresses: $($stat.AddressesInUse + $stat.AddressesFree)"
    Write-Host "  Addresses in Use: $($stat.AddressesInUse)"
    Write-Host "  Addresses Free: $($stat.AddressesFree)"
    Write-Host "  Percentage Used: $([math]::Round(($stat.AddressesInUse / ($stat.AddressesInUse + $stat.AddressesFree)) * 100, 2))%"
    Write-Host ""
}
```

#### Lease Information
```powershell
# Get current DHCP leases
$leases = Get-DhcpServerv4Lease -ComputerName "PDC.rev.local"

$leases | Select-Object IPAddress, HostName, ClientId, AddressState, LeaseExpiryTime | 
    Sort-Object IPAddress | Format-Table -AutoSize

# Export lease information
$leases | Export-Csv -Path "C:\Reports\DHCPLeases.csv" -NoTypeInformation
```

#### DHCP Server Performance
```powershell
# Get DHCP server performance counters
$perfCounters = Get-Counter -Counter "\DHCP Server\Discovers/sec", "\DHCP Server\Offers/sec", "\DHCP Server\Acks/sec", "\DHCP Server\Nacks/sec" -SampleInterval 5 -MaxSamples 3

$perfCounters | ForEach-Object {
    $_.CounterSamples | Select-Object Path, CookedValue | Format-Table -AutoSize
}
```

### DHCP Backup and Restore

#### Backup Configuration
```powershell
# Backup DHCP database
Backup-DhcpServer -ComputerName "PDC.rev.local" -Path "C:\DHCPBackup" -Force

# Schedule regular backups
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-Command 'Backup-DhcpServer -ComputerName PDC.rev.local -Path C:\DHCPBackup -Force'"
$trigger = New-ScheduledTaskTrigger -Daily -At 2am
$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable

Register-ScheduledTask -TaskName "DHCP Backup" -Action $action -Trigger $trigger -Settings $settings
```

#### Restore Configuration
```powershell
# Restore DHCP database (use with caution)
Restore-DhcpServer -ComputerName "PDC.rev.local" -Path "C:\DHCPBackup" -Force

# Restart DHCP service after restore
Restart-Service -Name DHCPServer -Force
```

## 🔧 Advanced DHCP Configuration

### DHCP Failover Configuration

#### Create Failover Relationship
```powershell
# Note: This requires a second DHCP server
# Example for future implementation

# Add failover partner server
Add-DhcpServerv4Failover -ComputerName "PDC.rev.local" -PartnerServer "PDC02.rev.local" -Name "PDC-PDC02-Failover" -SharedSecret "SecurePassword123" -MaxClientLeadTime 1:00:00 -StateActive -Force

# Configure failover scope
Add-DhcpServerv4FailoverScope -ComputerName "PDC.rev.local" -Name "PDC-PDC02-Failover" -ScopeId 192.168.1.64
```

### DHCP Policies

#### Create Vendor-Specific Policy
```powershell
# Create policy for HP printers
Add-DhcpServerv4Policy -ComputerName "PDC.rev.local" -Name "HP Printers" -Condition OR -VendorClass "Hewlett-Packard" -Description "Policy for HP printers"

# Set policy options
Set-DhcpServerv4OptionValue -ComputerName "PDC.rev.local" -PolicyName "HP Printers" -OptionId 3 -Value 192.168.1.65

# Assign policy to reservations
Get-DhcpServerv4Reservation -ComputerName "PDC.rev.local" | Where-Object {$_.Description -like "*HP*"} | 
    ForEach-Object {
        Add-DhcpServerv4PolicyIPRange -ComputerName "PDC.rev.local" -Name "HP Printers" -ScopeId 192.168.1.64 -StartRange $_.IPAddress -EndRange $_.IPAddress
    }
```

### DHCP Logging Configuration

#### Configure Logging
```powershell
# Enable DHCP audit logging
Set-DhcpServerv4AuditLog -ComputerName "PDC.rev.local" -Enable $true -DiskCheck 30 -MaxLogFiles 10

# Configure log path
Set-DhcpServerv4AuditLog -ComputerName "PDC.rev.local" -LogPath "C:\Windows\System32\DHCP"

# Analyze DHCP logs
function Get-DhcpLogAnalysis {
    param(
        [string]$LogPath = "C:\Windows\System32\DHCP"
    )
    
    $logFiles = Get-ChildItem -Path $LogPath -Filter "Dhcp*.log" | Sort-Object LastWriteTime -Descending | Select-Object -First 1
    
    if ($logFiles) {
        $logContent = Get-Content $logFiles.FullName
        
        # Analyze recent activity
        $recentActivity = $logContent | Where-Object {$_ -match ","} | 
            ConvertFrom-Csv -Header "ID","Date","Time","Description","IP Address","Host Name","MAC Address","User Name","TransactionID","QResult","Probationtime","CorrelationID","DnsResult"
        
        return $recentActivity
    } else {
        return "No DHCP log files found"
    }
}

# Get recent DHCP activity
$dhcpActivity = Get-DhcpLogAnalysis
$dhcpActivity | Select-Object -First 10 | Format-Table -AutoSize
```

## 🔍 DHCP Troubleshooting

### Common Issues and Solutions

#### DHCP Server Not Responding
```powershell
# Check DHCP service status
Get-Service -Name DHCPServer

# Check DHCP server authorization
Get-DhcpServerInDC

# Check network binding
Get-DhcpServerv4Binding

# Restart DHCP service
Restart-Service -Name DHCPServer -Force
```

#### IP Address Conflicts
```powershell
# Function to detect IP conflicts
function Test-IPConflict {
    param(
        [string]$IPAddress
    )
    
    # Ping the IP address
    $ping = Test-Connection -ComputerName $IPAddress -Count 2 -Quiet
    
    if ($ping) {
        # Get ARP table to find MAC address
        $arp = arp -a | Where-Object {$_ -match $IPAddress}
        return $arp
    } else {
        return "IP address $IPAddress is available"
    }
}

# Test specific IP for conflicts
Test-IPConflict -IPAddress "192.168.1.200"
```

#### Client Not Getting IP Address
```powershell
# Function to troubleshoot DHCP client issues
function Test-DhcpClient {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Release current IP
            ipconfig /release
            
            # Renew IP
            ipconfig /renew
            
            # Display new configuration
            ipconfig /all
            
            # Test connectivity
            Test-Connection -ComputerName "192.168.1.10" -Count 2
        }
        
        return $result
        
    } catch {
        return "Error troubleshooting DHCP client on $ComputerName`: $_"
    }
}

# Troubleshoot client DHCP issues
Test-DhcpClient -ComputerName "HR-PC-01"
```

## 📋 DHCP Validation Procedures

### Scope Validation
```powershell
# Function to validate DHCP configuration
function Test-DhcpConfiguration {
    param(
        [string]$ServerName = "PDC.rev.local"
    )
    
    $validationResults = @()
    
    # Test server authorization
    $authStatus = Get-DhcpServerInDC | Where-Object {$_.DnsName -eq $ServerName}
    $validationResults += [PSCustomObject]@{
        Test = "Server Authorization"
        Status = if ($authStatus) { "Passed" } else { "Failed" }
        Details = if ($authStatus) { "Server is authorized in AD" } else { "Server not authorized" }
    }
    
    # Test scope configuration
    $scopes = Get-DhcpServerv4Scope -ComputerName $ServerName
    $validationResults += [PSCustomObject]@{
        Test = "Scope Configuration"
        Status = if ($scopes.Count -gt 0) { "Passed" } else { "Failed" }
        Details = "Found $($scopes.Count) configured scopes"
    }
    
    # Test scope options
    foreach ($scope in $scopes) {
        $options = Get-DhcpServerv4OptionValue -ComputerName $ServerName -ScopeId $scope.ScopeId
        $validationResults += [PSCustomObject]@{
            Test = "Scope Options - $($scope.ScopeId)"
            Status = if ($options.Count -gt 0) { "Passed" } else { "Failed" }
            Details = "Found $($options.Count) configured options"
        }
    }
    
    return $validationResults
}

# Run DHCP validation
$validationResults = Test-DhcpConfiguration
$validationResults | Format-Table -AutoSize
```

### Performance Validation
```powershell
# Function to test DHCP performance
function Test-DhcpPerformance {
    $startTime = Get-Date
    
    # Get DHCP statistics
    $stats = Get-DhcpServerv4ScopeStatistics -ComputerName "PDC.rev.local"
    
    $endTime = Get-Date
    $duration = ($endTime - $startTime).TotalMilliseconds
    
    [PSCustomObject]@{
        Test = "DHCP Performance"
        ResponseTime = "$duration ms"
        ActiveScopes = $stats.Count
        TotalAddresses = ($stats | Measure-Object -Property AddressesInUse -Sum).Sum
        Status = if ($duration -lt 1000) { "Good" } else { "Slow" }
    }
}

# Test DHCP performance
Test-DhcpPerformance
```

## 🖼️ Screenshots

### DHCP Server Installation
![DHCP Server Role](../screenshots/04-network-services/dhcp-installation.png)

### Scope Configuration
![DHCP Scope Setup](../screenshots/04-network-services/dhcp-scope.png)

### DHCP Reservations
![DHCP Reservations](../screenshots/04-network-services/dhcp-reservations.png)

### DHCP Statistics
![DHCP Statistics](../screenshots/04-network-services/dhcp-statistics.png)

## 📋 DHCP Configuration Summary

| Component | Configuration | Status |
|------------|----------------|---------|
| DHCP Server | 192.168.1.10 (PDC) | ✅ Installed |
| Client Scope | 192.168.1.66-126/26 | ✅ Configured |
| Guest Scope | 192.168.1.130-239/25 | ✅ Configured |
| Exclusions | 192.168.1.80-85, 10-20, 240-254 | ✅ Configured |
| Reservations | HRPC01: 192.168.1.200 | ✅ Configured |
| DNS Integration | Dynamic Updates Enabled | ✅ Configured |
| Logging | Audit Logging Enabled | ✅ Configured |

### Key Features Implemented
- ✅ DHCP Server role installed and authorized
- ✅ Primary client scope (192.168.1.64/26) configured
- ✅ Guest network scope (192.168.1.128/25) configured
- ✅ Network device exclusions configured
- ✅ HR workstation reservation (192.168.1.200) configured
- ✅ DNS dynamic updates enabled
- ✅ Comprehensive monitoring and logging
- ✅ Backup and restore procedures established

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
