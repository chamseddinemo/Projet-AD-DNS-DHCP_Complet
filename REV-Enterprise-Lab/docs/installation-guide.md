# Installation Guide

## 📋 Description

This document provides step-by-step installation procedures for REV-Enterprise-Lab environment, including server setup, Active Directory installation, and client configuration.

## 🎯 Objectives

- Provide complete installation procedures
- Ensure proper server configuration
- Guide through Active Directory setup
- Establish client deployment procedures
- Document all installation requirements

## 🖥️ Prerequisites

### Hardware Requirements

#### Domain Controller (PDC)
- **CPU**: 4+ cores, 2.5+ GHz
- **Memory**: 16GB+ RAM
- **Storage**: 100GB+ SSD for OS, 500GB+ for data
- **Network**: Gigabit Ethernet adapter
- **Virtualization**: Hyper-V or VMware support

#### File Server (FS01)
- **CPU**: 4+ cores, 2.0+ GHz
- **Memory**: 8GB+ RAM
- **Storage**: 100GB+ SSD for OS, 1TB+ for data
- **Network**: Gigabit Ethernet adapter

#### Client Workstations
- **CPU**: 2+ cores, 2.0+ GHz
- **Memory**: 8GB+ RAM
- **Storage**: 256GB+ SSD
- **Network**: Gigabit Ethernet adapter

### Software Requirements

#### Server Software
- **Operating System**: Windows Server 2019/2022 Standard/Datacenter
- **Roles**: Active Directory Domain Services, DNS, DHCP, File Services
- **Features**: .NET Framework 4.8+, PowerShell 5.1+
- **Updates**: Latest Windows updates and security patches

#### Client Software
- **Operating System**: Windows 10/11 Professional/Enterprise
- **Updates**: Latest Windows updates and security patches
- **PowerShell**: PowerShell 5.1+ or PowerShell 7+

## 🔧 Server Installation

### Domain Controller Setup

#### Step 1: Server Installation
```powershell
# 1. Boot from Windows Server installation media
# 2. Select language, time, and keyboard preferences
# 3. Click "Install Now"
# 4. Select Windows Server 2022 Standard (Desktop Experience)
# 5. Accept license terms
# 6. Select "Custom: Install Windows only"
# 7. Select installation drive (SSD recommended)
# 8. Wait for installation to complete
# 9. Set administrator password on first boot
```

#### Step 2: Initial Server Configuration
```powershell
# Configure server name
Rename-Computer -NewName "PDC" -Restart

# Configure network adapter
Get-NetAdapter | Set-NetIPInterface -InterfaceAlias "Ethernet" -InterfaceMetric 10
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "192.168.1.10" -PrefixLength 24 -DefaultGateway "192.168.1.1"
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.1.10", "8.8.8.8"

# Configure time zone
Set-TimeZone -Id "Eastern Standard Time"

# Enable Remote Desktop
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0 -Type DWORD
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "UserAuthentication" -Value 1 -Type DWORD

# Enable PowerShell remoting
Enable-PSRemoting -Force
```

#### Step 3: Active Directory Installation
```powershell
# Install Active Directory Domain Services
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Install DNS Server
Install-WindowsFeature -Name DNS -IncludeManagementTools

# Install DHCP Server
Install-WindowsFeature -Name DHCP -IncludeManagementTools

# Restart server
Restart-Computer -Force
```

#### Step 4: Domain Creation
```powershell
# Create new Active Directory domain
Import-Module ADDSDeployment

Install-ADDSForest `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -DomainMode "WinThreshold" `
    -DomainName "rev.local" `
    -DomainNetbiosName "REV" `
    -ForestMode "WinThreshold" `
    -InstallDns:$true `
    -LogPath "C:\Windows\NTDS" `
    -NoRebootOnCompletion:$false `
    -SysvolPath "C:\Windows\SYSVOL" `
    -Force:$true

# Server will restart automatically
```

### File Server Setup

#### Step 1: Server Installation
```powershell
# 1. Boot from Windows Server installation media
# 2. Follow same installation steps as Domain Controller
# 3. Set server name to "FS01"
# 4. Configure network settings
Rename-Computer -NewName "FS01" -Restart

# Configure network
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "192.168.1.15" -PrefixLength 24 -DefaultGateway "192.168.1.1"
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.1.10"
```

#### Step 2: File Services Installation
```powershell
# Install File Services role
Install-WindowsFeature -Name FS-FileServer -IncludeManagementTools

# Install File Server Resource Manager
Install-WindowsFeature -Name FS-Resource-Manager -IncludeManagementTools

# Restart server
Restart-Computer -Force
```

#### Step 3: Domain Join
```powershell
# Join server to domain
Add-Computer -DomainName "rev.local" -Credential (Get-Credential) -Restart -Force
```

## 🌐 Network Services Configuration

### DHCP Server Configuration

#### Step 1: DHCP Server Authorization
```powershell
# Authorize DHCP server in Active Directory
Add-DhcpServerInDC -DnsName "FS01.rev.local" -IPAddress "192.168.1.15"
```

#### Step 2: DHCP Scope Configuration
```powershell
# Create primary scope for client computers
Add-DhcpServerv4Scope `
    -Name "Primary Client Scope" `
    -Description "Primary scope for client computers" `
    -StartRange "192.168.1.66" `
    -EndRange "192.168.1.126" `
    -SubnetMask "255.255.255.192" `
    -State Active

# Configure scope options
Set-DhcpServerv4OptionValue `
    -ComputerName "FS01.rev.local" `
    -ScopeId "192.168.1.64" `
    -DnsServer "192.168.1.10"

Set-DhcpServerv4OptionValue `
    -ComputerName "FS01.rev.local" `
    -ScopeId "192.168.1.64" `
    -Router "192.168.1.65"

Set-DhcpServerv4OptionValue `
    -ComputerName "FS01.rev.local" `
    -ScopeId "192.168.1.64" `
    -DomainName "rev.local"

# Create guest scope
Add-DhcpServerv4Scope `
    -Name "Guest Scope" `
    -Description "Scope for guest computers" `
    -StartRange "192.168.1.130" `
    -EndRange "192.168.1.238" `
    -SubnetMask "255.255.255.128" `
    -State Active

# Configure exclusions
Add-DhcpServerv4ExclusionRange `
    -ComputerName "FS01.rev.local" `
    -ScopeId "192.168.1.64" `
    -StartRange "192.168.1.80" `
    -EndRange "192.168.1.85"

Add-DhcpServerv4ExclusionRange `
    -ComputerName "FS01.rev.local" `
    -ScopeId "192.168.1.64" `
    -StartRange "192.168.1.10" `
    -EndRange "192.168.1.20"

# Create reservations
Add-DhcpServerv4Reservation `
    -ComputerName "FS01.rev.local" `
    -ScopeId "192.168.1.64" `
    -IPAddress "192.168.1.200" `
    -ClientId "00-15-5D-01-23-45" `
    -Name "HRPC01" `
    -Description "HR Computer 01"
```

### DNS Server Configuration

#### Step 1: DNS Zone Configuration
```powershell
# Create forward lookup zone
Add-DnsServerPrimaryZone `
    -ComputerName "PDC.rev.local" `
    -Name "rev.local" `
    -ReplicationScope "Forest" `
    -PassThru

# Create reverse lookup zone
Add-DnsServerPrimaryZone `
    -ComputerName "PDC.rev.local" `
    -NetworkID "192.168.1.0/24" `
    -ReplicationScope "Forest" `
    -PassThru
```

#### Step 2: DNS Record Configuration
```powershell
# Create A records for servers
Add-DnsServerResourceRecordA `
    -ComputerName "PDC.rev.local" `
    -ZoneName "rev.local" `
    -Name "PDC" `
    -IPv4Address "192.168.1.10" `
    -CreatePtr

Add-DnsServerResourceRecordA `
    -ComputerName "PDC.rev.local" `
    -ZoneName "rev.local" `
    -Name "FS01" `
    -IPv4Address "192.168.1.15" `
    -CreatePtr

# Create A records for web servers (round robin)
Add-DnsServerResourceRecordA `
    -ComputerName "PDC.rev.local" `
    -ZoneName "rev.local" `
    -Name "www" `
    -IPv4Address "192.168.1.8" `
    -CreatePtr

Add-DnsServerResourceRecordA `
    -ComputerName "PDC.rev.local" `
    -ZoneName "rev.local" `
    -Name "www" `
    -IPv4Address "192.168.1.9" `
    -CreatePtr

# Create A record for HR application
Add-DnsServerResourceRecordA `
    -ComputerName "PDC.rev.local" `
    -ZoneName "rev.local" `
    -Name "hrapp" `
    -IPv4Address "192.168.1.20" `
    -CreatePtr

# Configure forwarders
Add-DnsServerForwarder `
    -ComputerName "PDC.rev.local" `
    -IPAddress "8.8.8.8", "8.8.4.4" `
    -PassThru
```

## 📁 File Server Configuration

### Share Creation

#### Step 1: Create Folder Structure
```powershell
# Create base shares directory
New-Item -Path "D:\Shares" -ItemType Directory -Force

# Create departmental shares
New-Item -Path "D:\Shares\Public" -ItemType Directory -Force
New-Item -Path "D:\Shares\HR" -ItemType Directory -Force
New-Item -Path "D:\Shares\Sales" -ItemType Directory -Force
New-Item -Path "D:\Shares\IT" -ItemType Directory -Force

# Create subfolders for HR
New-Item -Path "D:\Shares\HR\Documents" -ItemType Directory -Force
New-Item -Path "D:\Shares\HR\Templates" -ItemType Directory -Force
New-Item -Path "D:\Shares\HR\Reports" -ItemType Directory -Force
New-Item -Path "D:\Shares\HR\Confidential" -ItemType Directory -Force

# Create subfolders for Sales
New-Item -Path "D:\Shares\Sales\Documents" -ItemType Directory -Force
New-Item -Path "D:\Shares\Sales\Templates" -ItemType Directory -Force
New-Item -Path "D:\Shares\Sales\Reports" -ItemType Directory -Force
New-Item -Path "D:\Shares\Sales\Presentations" -ItemType Directory -Force

# Create subfolders for IT
New-Item -Path "D:\Shares\IT\Tools" -ItemType Directory -Force
New-Item -Path "D:\Shares\IT\Scripts" -ItemType Directory -Force
New-Item -Path "D:\Shares\IT\Documentation" -ItemType Directory -Force
New-Item -Path "D:\Shares\IT\Backups" -ItemType Directory -Force
```

#### Step 2: Create Network Shares
```powershell
# Create shares
New-SmbShare -Name "Public" -Path "D:\Shares\Public" -ReadAccess "Everyone" -ChangeAccess "Authenticated Users"
New-SmbShare -Name "HR" -Path "D:\Shares\HR" -ReadAccess "HR-Group" -ChangeAccess "HR-Group"
New-SmbShare -Name "Sales" -Path "D:\Shares\Sales" -ReadAccess "Sales-Group" -ChangeAccess "Sales-Group"
New-SmbShare -Name "IT" -Path "D:\Shares\IT" -ReadAccess "IT-Group" -ChangeAccess "IT-Group"

# Configure share permissions
Grant-SmbShareAccess -Name "Public" -AccountName "Everyone" -AccessRight Full -Force
Grant-SmbShareAccess -Name "HR" -AccountName "HR-Group" -AccessRight Change -Force
Grant-SmbShareAccess -Name "Sales" -AccountName "Sales-Group" -AccessRight Change -Force
Grant-SmbShareAccess -Name "IT" -AccountName "IT-Group" -AccessRight Change -Force
```

#### Step 3: Configure NTFS Permissions
```powershell
# Configure NTFS permissions for HR share
$hrAcl = Get-Acl "D:\Shares\HR"
$hrAcl.SetAccessRuleProtection($true, $false)

# Add HR-Group with modify access
$hrRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "REV\HR-Group",
    "Modify",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$hrAcl.SetAccessRule($hrRule)

# Add Domain Admins with full control
$adminRule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "BUILTIN\Administrators",
    "FullControl",
    "ContainerInherit, ObjectInherit",
    "None",
    "Allow"
)
$hrAcl.SetAccessRule($adminRule)

Set-Acl "D:\Shares\HR" $hrAcl

# Similar configuration for Sales and IT shares
# (Repeat for Sales and IT with appropriate groups)
```

## 💻 Client Configuration

### Windows 10 Installation

#### Step 1: Client Installation
```powershell
# 1. Boot from Windows 10 installation media
# 2. Select language, time, and keyboard preferences
# 3. Click "Install Now"
# 4. Select Windows 10 Professional
# 5. Accept license terms
# 6. Select "Custom: Install Windows only"
# 7. Select installation drive
# 8. Wait for installation to complete
# 9. Configure initial settings
# 10. Create local administrator account
```

#### Step 2: Domain Join
```powershell
# Configure network settings
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.1.10"

# Join domain
Add-Computer -DomainName "rev.local" -Credential (Get-Credential) -Restart -Force
```

#### Step 3: Client Configuration
```powershell
# Configure time zone
Set-TimeZone -Id "Eastern Standard Time"

# Enable Remote Desktop
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server" -Name "fDenyTSConnections" -Value 0 -Type DWORD

# Install required software
# Microsoft Office, Adobe Reader, etc.
```

## 🔧 Post-Installation Configuration

### Active Directory Post-Installation

#### Step 1: Create OU Structure
```powershell
# Create departmental OUs
New-ADOrganizationalUnit -Name "Departments" -Path "DC=rev,DC=local"
New-ADOrganizationalUnit -Name "Human Resources" -Path "OU=Departments,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "Sales" -Path "OU=Departments,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "IT Department" -Path "OU=Departments,DC=rev,DC=local"

# Create computer OUs
New-ADOrganizationalUnit -Name "Computers" -Path "DC=rev,DC=local"
New-ADOrganizationalUnit -Name "HR Computers" -Path "OU=Computers,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "Sales Computers" -Path "OU=Computers,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "IT Computers" -Path "OU=Computers,DC=rev,DC=local"

# Create user OUs
New-ADOrganizationalUnit -Name "Users" -Path "DC=rev,DC=local"
New-ADOrganizationalUnit -Name "HR Users" -Path "OU=Users,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "Sales Users" -Path "OU=Users,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "IT Users" -Path "OU=Users,DC=rev,DC=local"

# Create group OUs
New-ADOrganizationalUnit -Name "Groups" -Path "DC=rev,DC=local"
New-ADOrganizationalUnit -Name "HR Groups" -Path "OU=Groups,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "Sales Groups" -Path "OU=Groups,DC=rev,DC=local"
New-ADOrganizationalUnit -Name "IT Groups" -Path "OU=Groups,DC=rev,DC=local"
```

#### Step 2: Create Groups
```powershell
# Create departmental groups
New-ADGroup -Name "HR-Group" -GroupScope Global -GroupCategory Security -Path "OU=HR Groups,OU=Groups,DC=rev,DC=local"
New-ADGroup -Name "Sales-Group" -GroupScope Global -GroupCategory Security -Path "OU=Sales Groups,OU=Groups,DC=rev,DC=local"
New-ADGroup -Name "IT-Group" -GroupScope Global -GroupCategory Security -Path "OU=IT Groups,OU=Groups,DC=rev,DC=local"

# Create administrative groups
New-ADGroup -Name "HR-Admins" -GroupScope Global -GroupCategory Security -Path "OU=HR Groups,OU=Groups,DC=rev,DC=local"
New-ADGroup -Name "Sales-Admins" -GroupScope Global -GroupCategory Security -Path "OU=Sales Groups,OU=Groups,DC=rev,DC=local"
New-ADGroup -Name "IT-Admins" -GroupScope Global -GroupCategory Security -Path "OU=IT Groups,OU=Groups,DC=rev,DC=local"
```

#### Step 3: Create Users
```powershell
# Create HR users
New-ADUser -Name "john.smith" -UserPrincipalName "john.smith@rev.local" -GivenName "John" -Surname "Smith" -DisplayName "John Smith" -Department "Human Resources" -Title "HR Manager" -Path "OU=HR Users,OU=Users,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "TempPassword123!") -Enabled $true
New-ADUser -Name "sarah.johnson" -UserPrincipalName "sarah.johnson@rev.local" -GivenName "Sarah" -Surname "Johnson" -DisplayName "Sarah Johnson" -Department "Human Resources" -Title "HR Specialist" -Path "OU=HR Users,OU=Users,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "TempPassword123!") -Enabled $true

# Create Sales users
New-ADUser -Name "mike.wilson" -UserPrincipalName "mike.wilson@rev.local" -GivenName "Mike" -Surname "Wilson" -DisplayName "Mike Wilson" -Department "Sales" -Title "Sales Manager" -Path "OU=Sales Users,OU=Users,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "TempPassword123!") -Enabled $true
New-ADUser -Name "lisa.chen" -UserPrincipalName "lisa.chen@rev.local" -GivenName "Lisa" -Surname "Chen" -DisplayName "Lisa Chen" -Department "Sales" -Title "Sales Representative" -Path "OU=Sales Users,OU=Users,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "TempPassword123!") -Enabled $true

# Create IT users
New-ADUser -Name "david.brown" -UserPrincipalName "david.brown@rev.local" -GivenName "David" -Surname "Brown" -DisplayName "David Brown" -Department "IT" -Title "IT Manager" -Path "OU=IT Users,OU=Users,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "TempPassword123!") -Enabled $true
New-ADUser -Name "jane.davis" -UserPrincipalName "jane.davis@rev.local" -GivenName "Jane" -Surname "Davis" -DisplayName "Jane Davis" -Department "IT" -Title "IT Specialist" -Path "OU=IT Users,OU=Users,DC=rev,DC=local" -AccountPassword (ConvertTo-SecureString "TempPassword123!") -Enabled $true

# Add users to groups
Add-ADGroupMember -Identity "HR-Group" -Members "john.smith", "sarah.johnson"
Add-ADGroupMember -Identity "Sales-Group" -Members "mike.wilson", "lisa.chen"
Add-ADGroupMember -Identity "IT-Group" -Members "david.brown", "jane.davis"
```

## ✅ Installation Validation

### Validation Checklist

#### Domain Controller Validation
- [ ] Server is online and responding
- [ ] Active Directory is running
- [ ] DNS is resolving names correctly
- [ ] DHCP is assigning IP addresses
- [ ] Time synchronization is working
- [ ] Event logs show no critical errors

#### File Server Validation
- [ ] Server is joined to domain
- [ ] File shares are accessible
- [ ] NTFS permissions are correct
- [ ] Quotas are configured
- [ ] Backup is configured

#### Client Validation
- [ ] Computers are joined to domain
- [ ] Users can authenticate
- [ ] Network drives are mapped
- [ ] Group policies are applied
- [ ] Applications are accessible

## 🔍 Troubleshooting

### Common Installation Issues

#### Domain Controller Issues
- **Problem**: Server won't promote to domain controller
- **Solution**: Check DNS configuration, ensure static IP, verify network connectivity
- **Command**: `Test-ComputerSecureChannel -Server "PDC.rev.local"`

#### File Server Issues
- **Problem**: Cannot join domain
- **Solution**: Verify DNS settings, check network connectivity, ensure time synchronization
- **Command**: `Test-Connection -ComputerName "PDC.rev.local"`

#### Client Issues
- **Problem**: Cannot authenticate to domain
- **Solution**: Check network connectivity, verify time synchronization, reset computer account
- **Command**: `Reset-ComputerMachinePassword -Server "PDC.rev.local"`

## 📋 Installation Summary

| Component | Status | Notes |
|-----------|---------|-------|
| Domain Controller | ✅ Complete | PDC.rev.local configured |
| File Server | ✅ Complete | FS01.rev.local configured |
| Network Services | ✅ Complete | DNS and DHCP configured |
| Client Configuration | ✅ Complete | All clients joined to domain |
| Active Directory | ✅ Complete | Users, groups, and OUs created |
| File Shares | ✅ Complete | Shares and permissions configured |

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
