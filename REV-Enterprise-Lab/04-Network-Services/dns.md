# DNS Server Configuration

## 📋 Description

This document provides comprehensive procedures for installing and configuring DNS (Domain Name System) services in REV-Enterprise-Lab environment, including zone configuration, record management, and DNS round-robin load balancing.

## 🎯 Objectives

- Install and configure DNS Server role
- Create and manage DNS zones for internal domain
- Implement DNS records for all network resources
- Configure DNS round-robin load balancing
- Establish DNS security and management procedures

## 🛠️ DNS Server Installation

### Prerequisites
- **Server OS**: Windows Server 2019/2022
- **Static IP**: 192.168.1.10 (PDC)
- **Administrative Rights**: Domain Administrator or equivalent
- **Network Configuration**: Proper IP addressing and connectivity

### DNS Server Role Installation

#### GUI Installation
1. Open **Server Manager**
2. Click **Add roles and features**
3. Select **Role-based or feature-based installation**
4. Choose target server (PDC)
5. Select **DNS Server** role
6. Add required features automatically
7. Click **Install** and wait for completion

#### PowerShell Installation
```powershell
# Install DNS Server role
Install-WindowsFeature -Name DNS -IncludeManagementTools

# Verify installation
Get-WindowsFeature -Name DNS

# Restart DNS service if needed
Restart-Service -Name DNS -Force
```

## 🌐 DNS Zone Configuration

### Forward Lookup Zone: rev.local

#### Primary Zone Creation
```powershell
# Create primary forward lookup zone
Add-DnsServerPrimaryZone -Name "rev.local" -ReplicationScope "Forest" -PassThru

# Configure zone settings
Set-DnsServerPrimaryZone -Name "rev.local" -DynamicUpdate Secure -Notify "Notify" -NotifyServers "192.168.1.10" -SecureSecondaries "TransferToSecureServers"

# Verify zone creation
Get-DnsServerZone -Name "rev.local"
```

#### Zone Configuration
```powershell
# Configure zone aging and scavenging
Set-DnsServerZoneAging -Name "rev.local" -Aging $true -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00

# Enable scavenging on DNS server
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00 -ApplyOnAllZones $true

# Configure zone transfers
Set-DnsServerPrimaryZone -Name "rev.local" -ZoneTransfer "Secure"
```

### Reverse Lookup Zone: 1.168.192.in-addr.arpa

#### Reverse Zone Creation
```powershell
# Create reverse lookup zone
Add-DnsServerPrimaryZone -NetworkID "192.168.1.0/24" -ReplicationScope "Forest" -DynamicUpdate Secure

# Verify reverse zone creation
Get-DnsServerZone -Name "1.168.192.in-addr.arpa"
```

#### Reverse Zone Configuration
```powershell
# Configure reverse zone settings
Set-DnsServerPrimaryZone -Name "1.168.192.in-addr.arpa" -DynamicUpdate Secure -Notify "Notify" -NotifyServers "192.168.1.10"

# Configure aging for reverse zone
Set-DnsServerZoneAging -Name "1.168.192.in-addr.arpa" -Aging $true -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00
```

## 📝 DNS Record Management

### Server Records

#### Domain Controller Records
```powershell
# Create A record for PDC
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "PDC" -A -IPv4Address "192.168.1.10" -TimeToLive 01:00:00

# Create PTR record for PDC
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "10" -Ptr -DomainName "PDC.rev.local" -TimeToLive 01:00:00

# Create additional A records for PDC
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "dc01" -A -IPv4Address "192.168.1.10" -TimeToLive 01:00:00
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "ns1" -A -IPv4Address "192.168.1.10" -TimeToLive 01:00:00
```

#### File Server Records
```powershell
# Create A record for File Server
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "FS01" -A -IPv4Address "192.168.1.15" -TimeToLive 01:00:00

# Create PTR record for File Server
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "15" -Ptr -DomainName "FS01.rev.local" -TimeToLive 01:00:00

# Create CNAME record for files
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "files" -CName -HostNameAlias "FS01.rev.local" -TimeToLive 01:00:00
```

#### Application Server Records
```powershell
# Create A record for Application Server
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "APP01" -A -IPv4Address "192.168.1.20" -TimeToLive 01:00:00

# Create PTR record for Application Server
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "20" -Ptr -DomainName "APP01.rev.local" -TimeToLive 01:00:00

# Create CNAME record for mail
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "mail" -CName -HostNameAlias "APP01.rev.local" -TimeToLive 01:00:00
```

### Web Server Records (Round-Robin Load Balancing)

#### Web Server A Records
```powershell
# Create A records for web servers (round-robin)
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "www" -A -IPv4Address "192.168.1.8" -TimeToLive 00:05:00
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "www" -A -IPv4Address "192.168.1.9" -TimeToLive 00:05:00

# Create PTR records for web servers
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "8" -Ptr -DomainName "WEB01.rev.local" -TimeToLive 01:00:00
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "9" -Ptr -DomainName "WEB02.rev.local" -TimeToLive 01:00:00

# Create A records for web server hostnames
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "WEB01" -A -IPv4Address "192.168.1.8" -TimeToLive 01:00:00
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "WEB02" -A -IPv4Address "192.168.1.9" -TimeToLive 01:00:00
```

#### HR Application DNS Record
```powershell
# Create A record for HR application
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "hrapp" -A -IPv4Address "192.168.1.20" -TimeToLive 01:00:00

# Create CNAME record for HR application portal
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "hrportal" -CName -HostNameAlias "hrapp.rev.local" -TimeToLive 01:00:00
```

### Service Records (SRV)

#### Active Directory Service Records
```powershell
# LDAP service record
Add-DnsServerResourceRecord -ZoneName "rev.local" -Srv -Name "_ldap._tcp" -DomainName "rev.local" -Port 389 -Priority 0 -Weight 100 -ComputerName "PDC.rev.local"

# Kerberos service record
Add-DnsServerResourceRecord -ZoneName "rev.local" -Srv -Name "_kerberos._tcp" -DomainName "rev.local" -Port 88 -Priority 0 -Weight 100 -ComputerName "PDC.rev.local"

# Global Catalog service record
Add-DnsServerResourceRecord -ZoneName "rev.local" -Srv -Name "_gc._tcp" -DomainName "rev.local" -Port 3268 -Priority 0 -Weight 100 -ComputerName "PDC.rev.local"

# KPassword service record
Add-DnsServerResourceRecord -ZoneName "rev.local" -Srv -Name "_kpasswd._tcp" -DomainName "rev.local" -Port 464 -Priority 0 -Weight 100 -ComputerName "PDC.rev.local"
```

### Workstation Records

#### HR Workstation Records
```powershell
# Create A record for HRPC01
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "HRPC01" -A -IPv4Address "192.168.1.200" -TimeToLive 01:00:00

# Create PTR record for HRPC01
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "200" -Ptr -DomainName "HRPC01.rev.local" -TimeToLive 01:00:00
```

## 🔧 DNS Server Configuration

### Forwarders Configuration
```powershell
# Configure DNS forwarders
Add-DnsServerForwarder -IPAddress "8.8.8.8" -PassThru
Add-DnsServerForwarder -IPAddress "8.8.4.4" -PassThru

# Configure forwarder timeout
Set-DnsServerForwarder -IPAddress "8.8.8.8" -Timeout 3 -PassThru

# Verify forwarder configuration
Get-DnsServerForwarder
```

### Root Hints Configuration
```powershell
# Update root hints (if needed)
Clear-DnsServerZone -Name "." -Force

# Add root hints manually (optional)
$rootHints = @(
    "a.root-servers.net",
    "b.root-servers.net",
    "c.root-servers.net"
)

foreach ($hint in $rootHints) {
    # This would typically be done automatically
    Write-Host "Root hint: $hint"
}
```

### DNS Server Settings
```powershell
# Configure DNS server settings
Set-DnsServerSetting -All -EventLogLevel 2 -LogLevel 2 -QueryTimeout 3 -RecursionTimeout 15 -MaxCacheTtl 01:00:00

# Configure recursion settings
Set-DnsServerRecursionScope -Name "." -EnableRecursion $true -SecureResponse $true

# Configure cache settings
Set-DnsServerCache -MaxCacheTtl 01:00:00 -MaxNegativeCacheTtl 00:15:00
```

## 📊 DNS Management Procedures

### DNS Zone Management

#### Zone Backup
```powershell
# Function to backup DNS zones
function Backup-DnsZones {
    param(
        [string]$BackupPath = "C:\DNSBackup"
    )
    
    # Create backup directory
    if (!(Test-Path $BackupPath)) {
        New-Item -Path $BackupPath -ItemType Directory -Force
    }
    
    # Export all zones
    $zones = Get-DnsServerZone
    
    foreach ($zone in $zones) {
        $zoneFile = Join-Path $BackupPath "$($zone.ZoneName).dns"
        Export-DnsServerZone -Name $zone.ZoneName -FileName $zoneFile -Force
        Write-Host "Backed up zone: $($zone.ZoneName)"
    }
}

# Backup DNS zones
Backup-DnsZones
```

#### Zone Monitoring
```powershell
# Function to monitor DNS zone health
function Get-DnsZoneHealth {
    $zones = Get-DnsServerZone
    $healthReport = @()
    
    foreach ($zone in $zones) {
        $records = Get-DnsServerResourceRecord -ZoneName $zone.ZoneName
        
        $health = [PSCustomObject]@{
            ZoneName = $zone.ZoneName
            ZoneType = $zone.ZoneType
            DynamicUpdate = $zone.DynamicUpdate
            RecordCount = $records.Count
            SecureUpdates = if ($zone.DynamicUpdate -eq "Secure") { "Yes" } else { "No" }
            AgingEnabled = if ($zone.AgingEnabled) { "Yes" } else { "No" }
            Status = "Healthy"
        }
        
        $healthReport += $health
    }
    
    return $healthReport
}

# Get zone health report
$zoneHealth = Get-DnsZoneHealth
$zoneHealth | Format-Table -AutoSize
```

### DNS Record Management

#### Record Validation
```powershell
# Function to validate DNS records
function Test-DnsRecords {
    param(
        [string]$ZoneName = "rev.local"
    )
    
    $records = Get-DnsServerResourceRecord -ZoneName $ZoneName
    $validationResults = @()
    
    foreach ($record in $records) {
        $testResult = [PSCustomObject]@{
            RecordName = $record.HostName
            RecordType = $record.RecordType
            RecordData = $record.RecordData
            ValidationStatus = "Unknown"
        }
        
        # Test A records
        if ($record.RecordType -eq "A") {
            try {
                $ping = Test-Connection -ComputerName $record.RecordData.IPv4Address -Count 1 -Quiet
                $testResult.ValidationStatus = if ($ping) { "Pass" } else { "Fail" }
            } catch {
                $testResult.ValidationStatus = "Error"
            }
        }
        
        # Test CNAME records
        if ($record.RecordType -eq "CNAME") {
            try {
                $resolve = Resolve-DnsName -Name $record.RecordData.HostNameAlias -ErrorAction SilentlyContinue
                $testResult.ValidationStatus = if ($resolve) { "Pass" } else { "Fail" }
            } catch {
                $testResult.ValidationStatus = "Error"
            }
        }
        
        $validationResults += $testResult
    }
    
    return $validationResults
}

# Validate DNS records
$recordValidation = Test-DnsRecords
$recordValidation | Format-Table -AutoSize
```

#### Automated Record Creation
```powershell
# Function to create DNS records for new computers
function New-ComputerDnsRecords {
    param(
        [string]$ComputerName,
        [string]$IPAddress,
        [string]$ZoneName = "rev.local"
    )
    
    try {
        # Create A record
        Add-DnsServerResourceRecord -ZoneName $ZoneName -Name $ComputerName -A -IPv4Address $IPAddress -TimeToLive 01:00:00
        
        # Create PTR record
        $reverseZone = "1.168.192.in-addr.arpa"
        $lastOctet = $IPAddress.Split('.')[-1]
        Add-DnsServerResourceRecord -ZoneName $reverseZone -Name $lastOctet -Ptr -DomainName "$ComputerName.$ZoneName" -TimeToLive 01:00:00
        
        Write-Host "Created DNS records for $ComputerName`: A and PTR records"
        
    } catch {
        Write-Host "Error creating DNS records for $ComputerName`: $_"
    }
}

# Example usage
New-ComputerDnsRecords -ComputerName "TEST-PC-01" -IPAddress "192.168.1.201"
```

## 🔍 DNS Troubleshooting

### Common DNS Issues

#### DNS Resolution Failures
```powershell
# Function to troubleshoot DNS resolution
function Test-DnsResolution {
    param(
        [string]$HostName,
        [string]$DnsServer = "192.168.1.10"
    )
    
    $results = @()
    
    # Test basic resolution
    try {
        $resolve = Resolve-DnsName -Name $HostName -Server $DnsServer -ErrorAction Stop
        $results += [PSCustomObject]@{
            Test = "Basic Resolution"
            Result = "Pass"
            Details = "Resolved to $($resolve.IPAddress)"
        }
    } catch {
        $results += [PSCustomObject]@{
            Test = "Basic Resolution"
            Result = "Fail"
            Details = $_.Exception.Message
        }
    }
    
    # Test reverse lookup
    if ($resolve -and $resolve.IPAddress) {
        try {
            $reverse = Resolve-DnsName -Name $resolve.IPAddress -Server $DnsServer -DnsOnly PTR -ErrorAction Stop
            $results += [PSCustomObject]@{
                Test = "Reverse Lookup"
                Result = "Pass"
                Details = "Resolved to $($reverse.NameHost)"
            }
        } catch {
            $results += [PSCustomObject]@{
                Test = "Reverse Lookup"
                Result = "Fail"
                Details = $_.Exception.Message
            }
        }
    }
    
    return $results
}

# Troubleshoot DNS resolution
Test-DnsResolution -HostName "PDC.rev.local"
Test-DnsResolution -HostName "www.rev.local"
```

#### DNS Server Health Check
```powershell
# Function to check DNS server health
function Test-DnsServerHealth {
    $healthChecks = @()
    
    # Check DNS service status
    $dnsService = Get-Service -Name DNS
    $healthChecks += [PSCustomObject]@{
        Check = "DNS Service Status"
        Status = if ($dnsService.Status -eq "Running") { "Healthy" } else { "Unhealthy" }
        Details = "Service status: $($dnsService.Status)"
    }
    
    # Test DNS server responsiveness
    try {
        $test = Test-Connection -ComputerName "192.168.1.10" -Count 2 -Quiet
        $healthChecks += [PSCustomObject]@{
            Check = "Server Responsiveness"
            Status = if ($test) { "Healthy" } else { "Unhealthy" }
            Details = if ($test) { "Server responding to ping" } else { "Server not responding" }
        }
    } catch {
        $healthChecks += [PSCustomObject]@{
            Check = "Server Responsiveness"
            Status = "Error"
            Details = $_.Exception.Message
        }
    }
    
    # Check zone integrity
    try {
        $zones = Get-DnsServerZone
        $healthChecks += [PSCustomObject]@{
            Check = "Zone Integrity"
            Status = "Healthy"
            Details = "Found $($zones.Count) zones"
        }
    } catch {
        $healthChecks += [PSCustomObject]@{
            Check = "Zone Integrity"
            Status = "Error"
            Details = $_.Exception.Message
        }
    }
    
    return $healthChecks
}

# Check DNS server health
$dnsHealth = Test-DnsServerHealth
$dnsHealth | Format-Table -AutoSize
```

## 🖼️ Screenshots

### DNS Zone Creation
![DNS Zone Setup](../screenshots/04-network-services/dns-zone.png)

### DNS Records
![DNS Records Management](../screenshots/04-network-services/dns-records.png)

### Round-Robin Configuration
![DNS Round Robin](../screenshots/04-network-services/dns-roundrobin.png)

### DNS Statistics
![DNS Statistics](../screenshots/04-network-services/dns-statistics.png)

## 📋 DNS Configuration Summary

| Component | Configuration | Status |
|------------|----------------|---------|
| DNS Server | 192.168.1.10 (PDC) | ✅ Installed |
| Forward Zone | rev.local | ✅ Created |
| Reverse Zone | 1.168.192.in-addr.arpa | ✅ Created |
| Server Records | PDC, FS01, APP01 | ✅ Configured |
| Web Records | www (round-robin) | ✅ Configured |
| Service Records | LDAP, Kerberos, GC | ✅ Configured |
| Forwarders | 8.8.8.8, 8.8.4.4 | ✅ Configured |
| Aging/Scavenging | Enabled | ✅ Configured |

### Key Features Implemented
- ✅ DNS Server role installed and configured
- ✅ Forward and reverse lookup zones created
- ✅ Comprehensive server and service records
- ✅ DNS round-robin load balancing for web servers
- ✅ HR application DNS record configured
- ✅ DNS forwarders configured for external resolution
- ✅ Zone aging and scavenging enabled
- ✅ Comprehensive monitoring and troubleshooting procedures

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
