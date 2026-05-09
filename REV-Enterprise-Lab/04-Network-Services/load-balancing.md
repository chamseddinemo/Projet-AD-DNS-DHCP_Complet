# Load Balancing Configuration

## 📋 Description

This document outlines the load balancing configuration implemented in REV-Enterprise-Lab environment, focusing on DNS round-robin load balancing for web services and future network load balancing strategies.

## 🎯 Objectives

- Implement DNS round-robin load balancing for web services
- Configure load distribution across multiple web servers
- Establish monitoring for load balancer performance
- Plan for future network load balancing implementation
- Ensure high availability for critical services

## 🌐 DNS Round-Robin Load Balancing

### Web Server Load Balancing

#### Current Web Server Configuration
```powershell
# Current web server configuration
$webServers = @(
    @{Name = "WEB01"; IP = "192.168.1.8"; Weight = 1; Status = "Active"},
    @{Name = "WEB02"; IP = "192.168.1.9"; Weight = 1; Status = "Active"}
)

# Display current configuration
$webServers | Format-Table -AutoSize
```

#### DNS Round-Robin Implementation
```powershell
# Create multiple A records for www.rev.local
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "www" -A -IPv4Address "192.168.1.8" -TimeToLive 00:05:00
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "www" -A -IPv4Address "192.168.1.9" -TimeToLive 00:05:00

# Verify round-robin records
Get-DnsServerResourceRecord -ZoneName "rev.local" -Name "www" | Where-Object {$_.RecordType -eq "A"}

# Create PTR records for web servers
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "8" -Ptr -DomainName "WEB01.rev.local" -TimeToLive 01:00:00
Add-DnsServerResourceRecord -ZoneName "1.168.192.in-addr.arpa" -Name "9" -Ptr -DomainName "WEB02.rev.local" -TimeToLive 01:00:00
```

#### Round-Robin Testing
```powershell
# Function to test DNS round-robin
function Test-DnsRoundRobin {
    param(
        [string]$HostName = "www.rev.local",
        [int]$TestCount = 10
    )
    
    $results = @()
    
    for ($i = 1; $i -le $TestCount; $i++) {
        try {
            $resolve = Resolve-DnsName -Name $HostName -DnsOnly -ErrorAction Stop
            $resolvedIP = $resolve.IPAddress[0]
            
            $results += [PSCustomObject]@{
                TestNumber = $i
                ResolvedIP = $resolvedIP
                ServerName = switch ($resolvedIP) {
                    "192.168.1.8" { "WEB01" }
                    "192.168.1.9" { "WEB02" }
                    default { "Unknown" }
                }
                Timestamp = Get-Date
            }
            
            # Small delay between tests
            Start-Sleep -Milliseconds 100
            
        } catch {
            $results += [PSCustomObject]@{
                TestNumber = $i
                ResolvedIP = "Error"
                ServerName = "Error"
                Timestamp = Get-Date
                Error = $_.Exception.Message
            }
        }
    }
    
    return $results
}

# Test round-robin resolution
$roundRobinTest = Test-DnsRoundRobin -TestCount 20
$roundRobinTest | Format-Table -AutoSize

# Analyze distribution
$ipDistribution = $roundRobinTest | Where-Object {$_.ResolvedIP -ne "Error"} | Group-Object ResolvedIP
$ipDistribution | Select-Object Name, Count | ForEach-Object {
    [PSCustomObject]@{
        IPAddress = $_.Name
        RequestCount = $_.Count
        Percentage = [math]::Round(($_.Count / $roundRobinTest.Count) * 100, 2)
    }
} | Format-Table -AutoSize
```

### Application Load Balancing

#### HR Application Load Balancing
```powershell
# Create multiple A records for HR application (future expansion)
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "hrapp" -A -IPv4Address "192.168.1.20" -TimeToLive 00:05:00

# For future: Add additional HR application servers
# Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "hrapp" -A -IPv4Address "192.168.1.21" -TimeToLive 00:05:00

# Create CNAME for HR application portal
Add-DnsServerResourceRecord -ZoneName "rev.local" -Name "hrportal" -CName -HostNameAlias "hrapp.rev.local" -TimeToLive 01:00:00
```

#### Service-Specific Load Balancing
```powershell
# Create load-balanced records for different services
$services = @(
    @{Name = "web"; IPs = @("192.168.1.8", "192.168.1.9"); TTL = "00:05:00"},
    @{Name = "api"; IPs = @("192.168.1.20"); TTL = "00:05:00"},  # Single server for now
    @{Name = "files"; IPs = @("192.168.1.15"); TTL = "01:00:00"}   # File server
)

foreach ($service in $services) {
    foreach ($ip in $service.IPs) {
        Add-DnsServerResourceRecord -ZoneName "rev.local" -Name $service.Name -A -IPv4Address $ip -TimeToLive $service.TTL
    }
    Write-Host "Configured load balancing for $($service.Name)"
}
```

## 🔧 Advanced Load Balancing Configuration

### Weighted Round-Robin (Future Implementation)

#### Planning for Weighted Load Balancing
```powershell
# Future weighted round-robin configuration
$weightedServers = @(
    @{Name = "WEB01"; IP = "192.168.1.8"; Weight = 2; Capacity = "High"},
    @{Name = "WEB02"; IP = "192.168.1.9"; Weight = 1; Capacity = "Medium"}
)

# This would require DNS server with weighted round-robin support
# or implementation of Windows Network Load Balancing

Write-Host "Planned weighted configuration:"
$weightedServers | Format-Table -AutoSize
```

### Health Check Configuration

#### DNS Health Monitoring
```powershell
# Function to monitor web server health
function Test-WebServerHealth {
    param(
        [array]$Servers = @("192.168.1.8", "192.168.1.9")
    )
    
    $healthResults = @()
    
    foreach ($server in $Servers) {
        $health = [PSCustomObject]@{
            ServerIP = $server
            ServerName = switch ($server) {
                "192.168.1.8" { "WEB01" }
                "192.168.1.9" { "WEB02" }
                default { "Unknown" }
            }
            HTTPStatus = "Unknown"
            ResponseTime = 0
            Status = "Unknown"
        }
        
        try {
            # Test HTTP connectivity
            $httpTest = Test-NetConnection -ComputerName $server -Port 80 -WarningAction SilentlyContinue
            
            if ($httpTest.TcpTestSucceeded) {
                $health.HTTPStatus = "Open"
                $health.Status = "Healthy"
                
                # Measure response time
                $startTime = Get-Date
                $webRequest = Invoke-WebRequest -Uri "http://$server" -TimeoutSec 5 -ErrorAction Stop
                $endTime = Get-Date
                $health.ResponseTime = [math]::Round(($endTime - $startTime).TotalMilliseconds, 2)
            } else {
                $health.HTTPStatus = "Closed"
                $health.Status = "Unhealthy"
            }
            
        } catch {
            $health.HTTPStatus = "Error"
            $health.Status = "Unhealthy"
            $health.ResponseTime = 0
        }
        
        $healthResults += $health
    }
    
    return $healthResults
}

# Monitor web server health
$serverHealth = Test-WebServerHealth
$serverHealth | Format-Table -AutoSize
```

#### Automated Health Checks
```powershell
# Function to perform automated health checks
function Invoke-LoadBalancerHealthCheck {
    $servers = @("192.168.1.8", "192.168.1.9")
    $unhealthyServers = @()
    
    foreach ($server in $servers) {
        try {
            $response = Invoke-WebRequest -Uri "http://$server/health" -TimeoutSec 10 -ErrorAction Stop
            
            if ($response.StatusCode -ne 200) {
                $unhealthyServers += $server
            }
        } catch {
            $unhealthyServers += $server
        }
    }
    
    # Remove unhealthy servers from DNS (manual process for now)
    if ($unhealthyServers.Count -gt 0) {
        Write-Host "Unhealthy servers detected: $($unhealthyServers -join ', ')"
        Write-Host "Manual intervention required to remove from DNS pool"
        
        # Log the event
        $eventMessage = "Load balancer detected unhealthy servers: $($unhealthyServers -join ', ')"
        Write-EventLog -LogName "Application" -Source "LoadBalancer" -EventId 1001 -EntryType Warning -Message $eventMessage
    }
    
    return $unhealthyServers
}

# Schedule regular health checks
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-Command 'Invoke-LoadBalancerHealthCheck'"
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date).AddMinutes(5) -RepetitionInterval (New-TimeSpan -Minutes 5)
$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopOnIdleEnd

Register-ScheduledTask -TaskName "LoadBalancer Health Check" -Action $action -Trigger $trigger -Settings $settings
```

## 📊 Load Balancing Monitoring

### Traffic Distribution Analysis

#### DNS Query Analysis
```powershell
# Function to analyze DNS query distribution
function Get-DnsQueryDistribution {
    param(
        [string]$LogPath = "C:\Windows\System32\dns"
    )
    
    # Analyze DNS query logs (if available)
    $logFiles = Get-ChildItem -Path $LogPath -Filter "*.log" | Sort-Object LastWriteTime -Descending | Select-Object -First 1
    
    if ($logFiles) {
        $logContent = Get-Content $logFiles.FullName
        
        # Filter queries for www.rev.local
        $wwwQueries = $logContent | Where-Object {$_ -match "www\.rev\.local"}
        
        # Analyze response distribution
        $distribution = @{}
        
        foreach ($query in $wwwQueries) {
            if ($query -match "192\.168\.1\.(8|9)") {
                $ip = $Matches[0]
                if ($distribution.ContainsKey($ip)) {
                    $distribution[$ip]++
                } else {
                    $distribution[$ip] = 1
                }
            }
        }
        
        return $distribution
    } else {
        return @{"No DNS logs available" = 0}
    }
}

# Analyze DNS query distribution
$queryDistribution = Get-DnsQueryDistribution
$queryDistribution.GetEnumerator() | ForEach-Object {
    [PSCustomObject]@{
        ServerIP = $_.Key
        QueryCount = $_.Value
        ServerName = switch ($_.Key) {
            "192.168.1.8" { "WEB01" }
            "192.168.1.9" { "WEB02" }
            default { "Unknown" }
        }
    }
} | Format-Table -AutoSize
```

### Performance Monitoring

#### Load Balancer Performance Metrics
```powershell
# Function to collect load balancer performance data
function Get-LoadBalancerPerformance {
    $servers = @("192.168.1.8", "192.168.1.9")
    $performanceData = @()
    
    foreach ($server in $servers) {
        try {
            # Get basic performance metrics
            $cpu = Get-Counter -ComputerName $server -Counter "\Processor(_Total)\% Processor Time" -SampleInterval 1 -MaxSamples 1
            $memory = Get-Counter -ComputerName $server -Counter "\Memory\Available MBytes" -SampleInterval 1 -MaxSamples 1
            $network = Get-Counter -ComputerName $server -Counter "\Network Interface(*)\Bytes Total/sec" -SampleInterval 1 -MaxSamples 1
            
            $performanceData += [PSCustomObject]@{
                ServerIP = $server
                ServerName = switch ($server) {
                    "192.168.1.8" { "WEB01" }
                    "192.168.1.9" { "WEB02" }
                    default { "Unknown" }
                }
                CPUUsage = [math]::Round($cpu.CounterSamples.CookedValue, 2)
                AvailableMemory = [math]::Round($memory.CounterSamples.CookedValue, 2)
                NetworkBytesSec = [math]::Round(($network.CounterSamples | Measure-Object -Property CookedValue -Sum).Sum, 2)
                Status = "Online"
                Timestamp = Get-Date
            }
            
        } catch {
            $performanceData += [PSCustomObject]@{
                ServerIP = $server
                ServerName = switch ($server) {
                    "192.168.1.8" { "WEB01" }
                    "192.168.1.9" { "WEB02" }
                    default { "Unknown" }
                }
                CPUUsage = 0
                AvailableMemory = 0
                NetworkBytesSec = 0
                Status = "Offline"
                Timestamp = Get-Date
            }
        }
    }
    
    return $performanceData
}

# Collect performance data
$performanceMetrics = Get-LoadBalancerPerformance
$performanceMetrics | Format-Table -AutoSize
```

## 🚀 Future Load Balancing Enhancements

### Network Load Balancing (NLB) Planning

#### NLB Cluster Configuration Plan
```powershell
# Future NLB cluster configuration
$nlbCluster = @{
    Name = "WEB-NLB-Cluster"
    VirtualIP = "192.168.1.100"
    Nodes = @(
        @{Name = "WEB01"; IP = "192.168.1.8"; Priority = 1},
        @{Name = "WEB02"; IP = "192.168.1.9"; Priority = 2}
    )
    Ports = @(80, 443)
    Mode = "Multicast"
    Affinity = "None"
}

Write-Host "Planned NLB Configuration:"
$nlbCluster | Format-List
```

#### NLB Implementation Steps (Future)
```powershell
# Future NLB implementation steps
$nlbSteps = @(
    "Install Network Load Balancing feature on all web servers",
    "Configure NLB cluster with virtual IP 192.168.1.100",
    "Add web servers as NLB nodes",
    "Configure port rules for HTTP (80) and HTTPS (443)",
    "Set affinity mode to None for optimal distribution",
    "Update DNS to point www.rev.local to 192.168.1.100",
    "Configure health monitoring for NLB cluster",
    "Test failover and load distribution"
)

Write-Host "Future NLB Implementation Plan:"
for ($i = 0; $i -lt $nlbSteps.Count; $i++) {
    Write-Host "$($i + 1). $($nlbSteps[$i])"
}
```

### Application Load Balancer Planning

#### Hardware Load Balancer Considerations
```powershell
# Future hardware load balancer requirements
$hardwareLB = @{
    Vendor = "F5 BIG-IP or Citrix ADC"
    Throughput = "1 Gbps"
    ConcurrentConnections = 10000
    SSLAcceleration = $true
    HealthMonitoring = $true
    PersistenceMethods = @("Source IP", "Cookie", "SSL Session")
    Features = @("SSL Offloading", "Compression", "Caching", "Web Application Firewall")
}

Write-Host "Future Hardware Load Balancer Requirements:"
$hardwareLB | Format-List
```

## 🔍 Load Balancing Troubleshooting

### Common Issues and Solutions

#### Uneven Load Distribution
```powershell
# Function to diagnose uneven load distribution
function Test-LoadDistribution {
    $servers = @("192.168.1.8", "192.168.1.9")
    $distribution = @()
    
    foreach ($server in $servers) {
        try {
            # Get current connection count (if available)
            $connections = Get-Counter -ComputerName $server -Counter "\Web Service(_Total)\Current Connections" -ErrorAction SilentlyContinue
            
            $distribution += [PSCustomObject]@{
                ServerIP = $server
                ServerName = switch ($server) {
                    "192.168.1.8" { "WEB01" }
                    "192.168.1.9" { "WEB02" }
                    default { "Unknown" }
                }
                CurrentConnections = if ($connections) { $connections.CounterSamples.CookedValue } else { 0 }
                Status = "Online"
            }
            
        } catch {
            $distribution += [PSCustomObject]@{
                ServerIP = $server
                ServerName = switch ($server) {
                    "192.168.1.8" { "WEB01" }
                    "192.168.1.9" { "WEB02" }
                    default { "Unknown" }
                }
                CurrentConnections = 0
                Status = "Error"
            }
        }
    }
    
    # Analyze distribution
    $totalConnections = ($distribution | Where-Object {$_.Status -eq "Online"} | Measure-Object -Property CurrentConnections -Sum).Sum
    
    foreach ($server in $distribution) {
        if ($server.Status -eq "Online" -and $totalConnections -gt 0) {
            $server | Add-Member -NotePropertyName "Percentage" -NotePropertyValue ([math]::Round(($server.CurrentConnections / $totalConnections) * 100, 2))
        } else {
            $server | Add-Member -NotePropertyName "Percentage" -NotePropertyValue 0
        }
    }
    
    return $distribution
}

# Test load distribution
$loadDistribution = Test-LoadDistribution
$loadDistribution | Format-Table -AutoSize
```

#### DNS Resolution Issues
```powershell
# Function to troubleshoot DNS round-robin issues
function Test-DnsRoundRobinTroubleshooting {
    $troubleshootingResults = @()
    
    # Test 1: Verify all A records exist
    $wwwRecords = Get-DnsServerResourceRecord -ZoneName "rev.local" -Name "www" | Where-Object {$_.RecordType -eq "A"}
    $troubleshootingResults += [PSCustomObject]@{
        Test = "DNS A Records"
        Result = if ($wwwRecords.Count -ge 2) { "Pass" } else { "Fail" }
        Details = "Found $($wwwRecords.Count) A records for www.rev.local"
    }
    
    # Test 2: Verify TTL settings
    $ttlCheck = $wwwRecords | Where-Object {$_.TimeToLive -le (New-TimeSpan -Minutes 10)}
    $troubleshootingResults += [PSCustomObject]@{
        Test = "TTL Settings"
        Result = if ($ttlCheck.Count -eq $wwwRecords.Count) { "Pass" } else { "Fail" }
        Details = "TTL should be 10 minutes or less for effective round-robin"
    }
    
    # Test 3: Test resolution multiple times
    $resolutionTest = Test-DnsRoundRobin -TestCount 10
    $uniqueIPs = ($resolutionTest | Where-Object {$_.ResolvedIP -ne "Error"} | Select-Object -ExpandProperty ResolvedIP | Sort-Object -Unique).Count
    $troubleshootingResults += [PSCustomObject]@{
        Test = "Resolution Distribution"
        Result = if ($uniqueIPs -ge 2) { "Pass" } else { "Fail" }
        Details = "Resolved to $uniqueIPs unique IP addresses out of 10 tests"
    }
    
    return $troubleshootingResults
}

# Run troubleshooting tests
$troubleshootingResults = Test-DnsRoundRobinTroubleshooting
$troubleshootingResults | Format-Table -AutoSize
```

## 🖼️ Screenshots

### DNS Round-Robin Configuration
![DNS Round Robin Setup](../screenshots/04-network-services/dns-roundrobin-config.png)

### Load Distribution Test
![Load Distribution](../screenshots/04-network-services/load-distribution.png)

### Health Monitoring
![Health Check Results](../screenshots/04-network-services/health-monitoring.png)

### Performance Metrics
![Performance Dashboard](../screenshots/04-network-services/performance-metrics.png)

## 📋 Load Balancing Summary

| Component | Configuration | Status |
|------------|----------------|---------|
| DNS Round-Robin | www.rev.local → 192.168.1.8, 192.168.1.9 | ✅ Configured |
| Web Servers | WEB01: 192.168.1.8, WEB02: 192.168.1.9 | ✅ Active |
| TTL Settings | 5 minutes for optimal distribution | ✅ Configured |
| Health Monitoring | Basic HTTP checks | ✅ Implemented |
| Performance Monitoring | CPU, Memory, Network metrics | ✅ Implemented |
| Future NLB | Planned for enhanced load balancing | 📋 Planned |
| Future Hardware LB | Considered for high availability | 📋 Planned |

### Key Features Implemented
- ✅ DNS round-robin load balancing for web services
- ✅ Multiple A records for www.rev.local
- ✅ Appropriate TTL settings for load distribution
- ✅ Health monitoring for web servers
- ✅ Performance metrics collection
- ✅ Troubleshooting procedures for common issues
- ✅ Future planning for NLB and hardware load balancers

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
