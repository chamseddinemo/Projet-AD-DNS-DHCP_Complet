# Active Directory Domain Setup

## 📋 Description

This document provides comprehensive procedures for installing and configuring Active Directory Domain Services (AD DS) for the REV-Enterprise-Lab environment, including domain controller promotion, forest configuration, and initial domain setup.

## 🎯 Objectives

- Install Active Directory Domain Services role
- Promote server to Primary Domain Controller
- Configure the rev.local domain
- Establish proper DNS integration
- Validate domain controller functionality
- Prepare domain for organizational unit structure

## 🛠️ Prerequisites

### System Requirements
- **Server OS**: Windows Server 2019/2022 Standard or Datacenter
- **Hardware**: Minimum 4GB RAM, 32GB storage, 2 CPU cores
- **Network**: Static IP configuration (192.168.1.10)
- **Administrative**: Domain administrator credentials

### Network Configuration
| Setting | Value | Description |
|---------|-------|-------------|
| IP Address | 192.168.1.10 | Static server IP |
| Subnet Mask | 255.255.255.192 | /26 network |
| Default Gateway | 192.168.1.1 | Network gateway |
| DNS Server | 127.0.0.1 | Point to self after promotion |
| Domain Name | rev.local | Internal domain name |

## 📋 Installation Procedure

### Step 1: Server Preparation

#### 1.1 Configure Network Settings
```powershell
# Set static IP address
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 26 -DefaultGateway 192.168.1.1

# Set DNS server (initially point to self)
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1

# Verify configuration
Get-NetIPConfiguration
```

#### 1.2 Configure Server Name
```powershell
# Set computer name
Rename-Computer -NewName "PDC" -Restart -Force
```

#### 1.3 Install Windows Updates
```powershell
# Install Windows Update module
Install-Module -Name PSWindowsUpdate -Force

# Install all available updates
Install-WindowsUpdate -AcceptAll -AutoReboot
```

### Step 2: Install Active Directory Domain Services

#### 2.1 GUI Installation
1. Open **Server Manager**
2. Click **Add roles and features**
3. Select **Role-based or feature-based installation**
4. Choose the target server
5. Select **Active Directory Domain Services** role
6. Add required features automatically
7. Click **Install** and wait for completion

#### 2.2 PowerShell Installation
```powershell
# Install AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Verify installation
Get-WindowsFeature -Name AD-Domain-Services
```

### Step 3: Promote to Domain Controller

#### 3.1 GUI Promotion
1. Open **Server Manager**
2. Click the notification flag for AD DS
3. Click **Promote this server to a domain controller**
4. Select **Add a new forest**
5. Configure domain settings:
   - **Root domain name**: rev.local
   - **Forest functional level**: Windows Server 2016
   - **Domain functional level**: Windows Server 2016
   - **Domain NetBIOS name**: REV
6. Set DSRM password
7. Verify DNS options (DNS Server will be installed automatically)
8. Review configuration and click **Install**

#### 3.2 PowerShell Promotion
```powershell
# Import AD module
Import-Module ADDSDeployment

# Promote to domain controller
Install-ADDSForest `
    -DomainName "rev.local" `
    -DomainNetbiosName "REV" `
    -ForestMode "Win2016" `
    -DomainMode "Win2016" `
    -DatabasePath "C:\Windows\NTDS" `
    -LogPath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -InstallDns:$true `
    -CreateDnsDelegation:$false `
    -NoRebootOnCompletion:$false `
    -Force:$true

# Set DSRM password (will be prompted)
```

### Step 4: Post-Installation Configuration

#### 4.1 Verify Domain Controller Status
```powershell
# Check domain controller status
Get-ADDomainController -Filter *

# Verify domain information
Get-ADDomain

# Check forest information
Get-ADForest

# Verify DNS configuration
Get-DnsServerZone
```

#### 4.2 Configure DNS Settings
```powershell
# Configure forwarders
Add-DnsServerForwarder -IPAddress 8.8.8.8,8.8.4.4

# Set aging/scavenging settings
Set-DnsServerZoneAging -Name "rev.local" -Aging $true -NoRefreshInterval 7.00:00:00 -RefreshInterval 7.00:00:00

# Enable scavenging
Set-DnsServerScavenging -ScavengingState $true -ScavengingInterval 7.00:00:00
```

#### 4.3 Configure Time Services
```powershell
# Configure NTP settings
w32tm /config /manualpeerlist:pool.ntp.org /syncfromflags:manual /reliable:yes /update

# Start time service
net start w32time

# Force time sync
w32tm /resync
```

## 🔧 Domain Configuration

### Forest and Domain Settings

| Setting | Value | Description |
|---------|-------|-------------|
| Forest Name | rev.local | Root forest domain |
| Domain Name | rev.local | Primary domain |
| NetBIOS Name | REV | Legacy compatibility |
| Forest Functional Level | Windows Server 2016 | Forest capabilities |
| Domain Functional Level | Windows Server 2016 | Domain capabilities |

### DNS Configuration

| Zone | Type | Dynamic Updates | Aging |
|------|------|------------------|-------|
| rev.local | Primary AD-Integrated | Secure | Enabled |
| 1.168.192.in-addr.arpa | Primary AD-Integrated | Secure | Enabled |
| _msdcs.rev.local | AD-Integrated | Secure | N/A |

## 📊 Domain Controller Validation

### Service Validation
```powershell
# Check essential services
Get-Service -Name "NTDS", "DNS", "Netlogon", "KDC" | Select-Object Name, Status

# Test domain controller functionality
Test-ADDSForestInstallation -DomainName "rev.local" -NoRebootOnCompletion

# Verify replication (single DC - should show no errors)
Get-ADReplicationPartnerMetadata -Target "PDC.rev.local"
```

### DNS Validation
```powershell
# Test DNS resolution
nslookup PDC.rev.local
nslookup localhost

# Check DNS records
Get-DnsServerResourceRecord -ZoneName "rev.local" | Where-Object {$_.HostName -eq "PDC"}

# Test SRV records
nslookup -type=srv _ldap._tcp.rev.local
nslookup -type=srv _kerberos._tcp.rev.local
```

### Authentication Validation
```powershell
# Test domain join capability
Test-ComputerSecureChannel -Server "PDC.rev.local"

# Check domain trust relationships
Get-ADTrust -Filter *

# Verify domain controller locator
nltest /dsgetdc:rev.local
```

## 🔒 Security Configuration

### Default Security Groups
| Group | Purpose | Default Members |
|-------|---------|-----------------|
| Domain Admins | Full domain administration | Administrator |
| Enterprise Admins | Forest-wide administration | Administrator |
| Schema Admins | Schema modifications | Administrator |
| Domain Users | Standard user accounts | All domain users |
| Domain Computers | Domain-joined computers | All domain computers |

### Default User Accounts
| Account | Purpose | Status |
|---------|---------|--------|
| Administrator | Built-in admin account | Enabled (rename recommended) |
| Guest | Built-in guest account | Disabled |
| krbtgt | Kerberos service account | Disabled by default |

## 📋 Maintenance Procedures

### Domain Controller Backup
```powershell
# Create system state backup
wbadmin start systemstatebackup -backupTarget:D:\Backup -quiet

# Schedule regular backups
schtasks /create /tn "ADBackup" /tr "wbadmin start systemstatebackup -backupTarget:D:\Backup -quiet" /sc weekly /d SUN /st 02:00
```

### Health Monitoring
```powershell
# Check domain controller health
dcdiag /v

# Check DNS health
dnslint /d rev.local

# Check replication health
repadmin /showrepl

# Check event logs for errors
Get-WinEvent -LogName "Directory Service" -MaxEvents 50 | Where-Object {$_.LevelDisplayName -eq "Error"}
```

## 🚀 Troubleshooting

### Common Issues and Solutions

#### Domain Controller Promotion Fails
**Symptoms**: Promotion wizard fails with various errors
**Solutions**:
- Verify network connectivity
- Check DNS resolution
- Ensure sufficient disk space
- Validate system time synchronization
- Check Windows Firewall settings

#### DNS Resolution Issues
**Symptoms**: Unable to resolve domain names
**Solutions**:
- Verify DNS service is running
- Check forwarder configuration
- Validate zone transfers
- Test network connectivity
- Review DNS event logs

#### Authentication Failures
**Symptoms**: Users cannot authenticate
**Solutions**:
- Check Netlogon service status
- Verify secure channel
- Test time synchronization
- Review security event logs
- Validate user account status

### Diagnostic Commands
```powershell
# Comprehensive domain controller test
dcdiag /c /e /v

# DNS diagnostics
dnslint /d rev.local /s localhost

# Network diagnostics
nltest /dsgetdc:rev.local
nltest /dclist:rev.local

# Time synchronization
w32tm /query /status
w32tm /query /source

# Event log analysis
Get-EventLog -LogName "Directory Service" -Newest 50 | Where-Object {$_.EntryType -eq "Error"}
```

## 🖼️ Screenshots

### Domain Controller Installation
![AD DS Installation](../screenshots/02-active-directory/ads-installation.png)

### Domain Promotion Wizard
![Promotion Wizard](../screenshots/02-active-directory/promotion-wizard.png)

### Post-Installation Validation
![Domain Controller Status](../screenshots/02-active-directory/dc-validation.png)

### DNS Configuration
![DNS Zones](../screenshots/02-active-directory/dns-zones.png)

### Domain Controller Tools
![AD Administrative Center](../screenshots/02-active-directory/ad-admin-center.png)

## 📊 Configuration Summary

| Component | Setting | Status |
|-----------|---------|--------|
| Domain Name | rev.local | ✅ Configured |
| Domain Controller | PDC.rev.local | ✅ Operational |
| DNS Service | Integrated | ✅ Running |
| Forest Functional Level | Windows Server 2016 | ✅ Set |
| Domain Functional Level | Windows Server 2016 | ✅ Set |
| Time Synchronization | NTP | ✅ Configured |
| Backup Schedule | Weekly | ✅ Configured |

## 🔄 Next Steps

1. Create organizational unit structure
2. Create user and group accounts
3. Configure group policies
4. Set up additional domain controllers
5. Implement monitoring and alerting
6. Document administrative procedures

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
