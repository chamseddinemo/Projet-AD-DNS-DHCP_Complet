# Windows 10 Domain Join

## 📋 Description

This document provides comprehensive procedures for joining Windows 10 workstations to the REV-Enterprise-Lab Active Directory domain, including domain join configuration, validation, and troubleshooting.

## 🎯 Objectives

- Join Windows 10 workstations to rev.local domain
- Configure proper network settings for domain join
- Validate domain membership and authentication
- Troubleshoot common domain join issues
- Ensure proper computer account configuration

## 🔧 Domain Join Prerequisites

### System Requirements
- **Operating System**: Windows 10 Professional or Enterprise
- **Network Connectivity**: Connection to domain network
- **DNS Configuration**: Proper DNS server settings
- **Administrative Rights**: Local administrator credentials
- **Domain Controller**: PDC.rev.local (192.168.1.10) must be reachable

### Network Configuration
```powershell
# Verify network configuration
Get-NetIPConfiguration | Format-Table -AutoSize

# Test DNS resolution
Test-Connection -ComputerName "PDC.rev.local" -Count 2

# Test domain controller connectivity
Test-NetConnection -ComputerName "192.168.1.10" -Port 389
Test-NetConnection -ComputerName "192.168.1.10" -Port 88
```

### Pre-Join Checklist
- [ ] Computer is connected to the network
- [ ] DNS server is set to 192.168.1.10
- [ ] Domain controller is reachable
- [ ] Local administrator account is available
- [ ] Computer name follows naming convention
- [ ] No pending Windows updates
- [ ] Antivirus software is compatible with domain join

## 🖥️ Domain Join Procedures

### Method 1: GUI Domain Join

#### Step-by-Step GUI Process
1. **Open System Properties**
   - Right-click Start button
   - Select "System"
   - Click "Change settings" next to "Computer name"

2. **Change Computer Name/Domain**
   - Click "Change" button
   - Select "Domain" option
   - Enter domain name: `rev.local`
   - Click "OK"

3. **Authentication Prompt**
   - Enter domain administrator credentials:
     - Username: `rev.local\administrator`
     - Password: `[Domain Admin Password]`
   - Click "OK"

4. **Restart Computer**
   - Click "OK" on welcome message
   - Click "Restart Now"

#### PowerShell GUI Method
```powershell
# Open System Properties using PowerShell
Start-Process sysdm.cpl

# Alternative: Use SystemPropertiesAdvanced.exe
Start-Process SystemPropertiesAdvanced.exe
```

### Method 2: PowerShell Domain Join

#### PowerShell Command
```powershell
# Join domain using PowerShell
Add-Computer -DomainName "rev.local" -Credential (Get-Credential) -Restart -Force

# Example with specific credentials
$credential = Get-Credential -UserName "rev.local\administrator" -Message "Enter domain administrator credentials"
Add-Computer -DomainName "rev.local" -Credential $credential -Restart -Force
```

#### PowerShell with Computer Name
```powershell
# Set computer name and join domain
Rename-Computer -NewName "HR-PC-01" -DomainName "rev.local" -Credential (Get-Credential) -Restart -Force

# Example for HR workstation
$credential = Get-Credential -UserName "rev.local\administrator" -Message "Enter domain administrator credentials"
Rename-Computer -NewName "HR-PC-01" -DomainName "rev.local" -Credential $credential -Restart -Force
```

### Method 3: Command Line Domain Join

#### Netdom Command
```cmd
# Join domain using netdom
netdom join %computername% /domain:rev.local /userd:rev.local\administrator /passwordd:[Password] /reboot

# Join with specific computer name
netdom join HR-PC-01 /domain:rev.local /userd:rev.local\administrator /passwordd:[Password] /reboot
```

#### PowerShell Alternative
```powershell
# Join domain using .NET method
$computer = Get-WmiObject -Class Win32_ComputerSystem
$computer.JoinDomainOrWorkgroup("rev.local", $null, "rev.local\administrator", "[Password]", $null)
```

## 🔍 Domain Join Validation

### Post-Join Verification

#### Check Domain Membership
```powershell
# Verify domain join
Get-WmiObject -Class Win32_ComputerSystem | Select-Object Name, Domain, DomainRole, PartOfDomain

# Alternative method
$computerInfo = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
Write-Host "Current domain: $($computerInfo.Name)"
```

#### Test Domain Authentication
```powershell
# Test domain authentication
Test-ComputerSecureChannel -Server "PDC.rev.local"

# Test user authentication
$credential = Get-Credential -UserName "rev.local\john.smith" -Message "Enter user credentials"
Test-ComputerSecureChannel -Server "PDC.rev.local" -Credential $credential
```

#### Verify DNS Registration
```powershell
# Check if computer is registered in DNS
$computerName = $env:COMPUTERNAME
$dnsRecord = Resolve-DnsName -Name "$computerName.rev.local" -ErrorAction SilentlyContinue

if ($dnsRecord) {
    Write-Host "Computer is registered in DNS"
    $dnsRecord | Format-Table -AutoSize
} else {
    Write-Host "Computer is NOT registered in DNS"
}
```

#### Check Group Policy Application
```powershell
# Force Group Policy update
gpupdate /force

# Check Group Policy results
gpresult /r /scope computer
gpresult /r /scope user
```

## 🔧 Post-Join Configuration

### Local Administrator Configuration

#### Add IT-Group as Local Administrator
```powershell
# Add IT-Group to local administrators
Add-LocalGroupMember -Group "Administrators" -Member "REV\IT-Group" -ErrorAction SilentlyContinue

# Verify group membership
Get-LocalGroupMember -Group "Administrators" | Where-Object {$_.ObjectClass -eq "Group"}
```

#### Configure Local Policies
```powershell
# Configure local security policies
secpol.msc

# Configure User Account Control (UAC)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "EnableLUA" -Value 1 -Type DWord
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value 2 -Type DWord
```

### Network Configuration

#### Configure Network Adapter
```powershell
# Get network adapters
Get-NetAdapter | Format-Table -AutoSize

# Set DNS servers
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.1.10"

# Configure network profile
Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory DomainAuthenticated
```

#### Configure Windows Firewall
```powershell
# Enable domain firewall profile
Set-NetFirewallProfile -Profile Domain -Enabled True

# Configure firewall rules for domain
New-NetFirewallRule -DisplayName "Allow Domain Traffic" -Direction Inbound -Protocol Any -Action Allow -Profile Domain
```

## 📊 Domain Join Automation

### Batch Domain Join Script
```batch
@echo off
REM Domain Join Script for REV Enterprise Lab

echo ========================================
echo REV Enterprise Lab - Domain Join
echo ========================================
echo.

REM Check if already joined to domain
wmic computersystem get domain | findstr /i "rev.local"
if %errorlevel% equ 0 (
    echo Computer is already joined to rev.local domain
    goto end
)

echo Joining computer to rev.local domain...

REM Set computer name based on user input
set /p computer_name=Enter computer name (e.g., HR-PC-01): 
if "%computer_name%"=="" (
    echo No computer name provided. Exiting.
    goto end
)

REM Rename computer
wmic computersystem where name="%computername%" rename name="%computer_name%"

REM Join domain
netdom join %computer_name% /domain:rev.local /userd:rev.local\administrator /passwordd:DomainAdminPassword /reboot

if %errorlevel% equ 0 (
    echo Domain join successful. Computer will restart.
) else (
    echo Domain join failed. Error code: %errorlevel%
)

:end
pause
```

### PowerShell Domain Join Script
```powershell
# PowerShell Domain Join Script
param(
    [Parameter(Mandatory=$true)]
    [string]$ComputerName,
    
    [Parameter(Mandatory=$true)]
    [string]$Domain = "rev.local",
    
    [Parameter(Mandatory=$true)]
    [string]$DomainUser = "rev.local\administrator"
)

Write-Host "REV Enterprise Lab - Domain Join Script" -ForegroundColor Green
Write-Host "=========================================" -ForegroundColor Green

# Check if already joined to domain
$currentDomain = (Get-WmiObject -Class Win32_ComputerSystem).Domain
if ($currentDomain -eq $Domain) {
    Write-Host "Computer is already joined to $Domain domain" -ForegroundColor Yellow
    return
}

Write-Host "Joining computer to $Domain domain..." -ForegroundColor Cyan

try {
    # Get domain administrator credentials
    $credential = Get-Credential -UserName $DomainUser -Message "Enter domain administrator credentials"
    
    # Rename computer and join domain
    Rename-Computer -NewName $ComputerName -DomainName $Domain -Credential $credential -Restart -Force -ErrorAction Stop
    
    Write-Host "Domain join successful. Computer will restart." -ForegroundColor Green
    Write-Host "New computer name: $ComputerName" -ForegroundColor Green
    Write-Host "Domain: $Domain" -ForegroundColor Green
    
} catch {
    Write-Host "Domain join failed: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host "Error details: $($_.Exception.ToString())" -ForegroundColor Red
}
```

## 🔍 Domain Join Troubleshooting

### Common Issues and Solutions

#### DNS Resolution Issues
```powershell
# Function to troubleshoot DNS issues
function Test-DNSConfiguration {
    $troubleshooting = @()
    
    # Test 1: Check DNS server settings
    $dnsSettings = Get-DnsClientServerAddress
    $troubleshooting += [PSCustomObject]@{
        Test = "DNS Server Configuration"
        Status = if ($dnsSettings.ServerAddresses -contains "192.168.1.10") { "Correct" } else { "Incorrect" }
        Details = "DNS servers: $($dnsSettings.ServerAddresses -join ', ')"
    }
    
    # Test 2: Test domain controller resolution
    try {
        $dcResolution = Resolve-DnsName -Name "PDC.rev.local" -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "DC Resolution"
            Status = "Success"
            Details = "PDC.rev.local resolves to $($dcResolution.IPAddress)"
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "DC Resolution"
            Status = "Failed"
            Details = "Cannot resolve PDC.rev.local: $($_.Exception.Message)"
        }
    }
    
    # Test 3: Test SRV record resolution
    try {
        $ldapRecord = Resolve-DnsName -Name "_ldap._tcp.rev.local" -Type SRV -ErrorAction Stop
        $troubleshooting += [PSCustomObject]@{
            Test = "SRV Record Resolution"
            Status = "Success"
            Details = "LDAP SRV record found: $($ldapRecord.NameTarget)"
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "SRV Record Resolution"
            Status = "Failed"
            Details = "Cannot resolve LDAP SRV record: $($_.Exception.Message)"
        }
    }
    
    return $troubleshooting
}

# Run DNS troubleshooting
$dnsTest = Test-DNSConfiguration
$dnsTest | Format-Table -AutoSize
```

#### Network Connectivity Issues
```powershell
# Function to troubleshoot network connectivity
function Test-NetworkConnectivity {
    $troubleshooting = @()
    
    # Test 1: Basic connectivity to DC
    $pingResult = Test-Connection -ComputerName "192.168.1.10" -Count 2
    $troubleshooting += [PSCustomObject]@{
        Test = "Basic Connectivity"
        Status = if ($pingResult) { "Success" } else { "Failed" }
        Details = "Ping to 192.168.1.10: $(if ($pingResult) { 'Success' } else { 'Failed' })"
    }
    
    # Test 2: LDAP port connectivity
    $ldapTest = Test-NetConnection -ComputerName "192.168.1.10" -Port 389
    $troubleshooting += [PSCustomObject]@{
        Test = "LDAP Port (389)"
        Status = if ($ldapTest.TcpTestSucceeded) { "Open" } else { "Closed" }
        Details = "LDAP port 389: $(if ($ldapTest.TcpTestSucceeded) { 'Open' } else { 'Closed' })"
    }
    
    # Test 3: Kerberos port connectivity
    $kerberosTest = Test-NetConnection -ComputerName "192.168.1.10" -Port 88
    $troubleshooting += [PSCustomObject]@{
        Test = "Kerberos Port (88)"
        Status = if ($kerberosTest.TcpTestSucceeded) { "Open" } else { "Closed" }
        Details = "Kerberos port 88: $(if ($kerberosTest.TcpTestSucceeded) { 'Open' } else { 'Closed' })"
    }
    
    # Test 4: SMB port connectivity
    $smbTest = Test-NetConnection -ComputerName "192.168.1.10" -Port 445
    $troubleshooting += [PSCustomObject]@{
        Test = "SMB Port (445)"
        Status = if ($smbTest.TcpTestSucceeded) { "Open" } else { "Closed" }
        Details = "SMB port 445: $(if ($smbTest.TcpTestSucceeded) { 'Open' } else { 'Closed' })"
    }
    
    return $troubleshooting
}

# Run network troubleshooting
$networkTest = Test-NetworkConnectivity
$networkTest | Format-Table -AutoSize
```

#### Authentication Issues
```powershell
# Function to troubleshoot authentication issues
function Test-DomainAuthentication {
    $troubleshooting = @()
    
    # Test 1: Check secure channel
    try {
        $secureChannel = Test-ComputerSecureChannel -Server "PDC.rev.local"
        $troubleshooting += [PSCustomObject]@{
            Test = "Secure Channel"
            Status = if ($secureChannel) { "Healthy" } else { "Broken" }
            Details = "Secure channel to PDC.rev.local: $(if ($secureChannel) { 'Healthy' } else { 'Broken' })"
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "Secure Channel"
            Status = "Error"
            Details = "Error testing secure channel: $($_.Exception.Message)"
        }
    }
    
    # Test 2: Check time synchronization
    $timeDiff = Get-WmiObject -Class Win32_TimeZone | Select-Object Bias
    $troubleshooting += [PSCustomObject]@{
        Test = "Time Synchronization"
        Status = "Manual Check Required"
        Details = "Time zone: $($timeDiff.Bias) minutes from UTC"
    }
    
    # Test 3: Check computer account status
    try {
        $computerAccount = Get-ADComputer -Identity $env:COMPUTERNAME -Properties Enabled, LastLogonDate
        $troubleshooting += [PSCustomObject]@{
            Test = "Computer Account"
            Status = if ($computerAccount.Enabled) { "Enabled" } else { "Disabled" }
            Details = "Account status: $($computerAccount.Enabled), Last logon: $($computerAccount.LastLogonDate)"
        }
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "Computer Account"
            Status = "Error"
            Details = "Error checking computer account: $($_.Exception.Message)"
        }
    }
    
    return $troubleshooting
}

# Run authentication troubleshooting
$authTest = Test-DomainAuthentication
$authTest | Format-Table -AutoSize
```

## 🖼️ Screenshots

### Domain Join Dialog
![Domain Join Wizard](../screenshots/06-client-configuration/domain-join-wizard.png)

### System Properties
![System Properties](../screenshots/06-client-configuration/system-properties.png)

### Domain Join Success
![Join Success](../screenshots/06-client-configuration/domain-join-success.png)

### Group Policy Update
![GPUpdate](../screenshots/06-client-configuration/gpupdate.png)

## 📋 Domain Join Summary

| Step | Action | Command | Status |
|------|--------|----------|---------|
| 1 | Network Configuration | Set DNS to 192.168.1.10 | ✅ Complete |
| 2 | Domain Join | Add-Computer -DomainName rev.local | ✅ Complete |
| 3 | Computer Restart | Required for domain join | ✅ Complete |
| 4 | Validation | Test-ComputerSecureChannel | ✅ Complete |
| 5 | Local Admin Setup | Add IT-Group to Administrators | ✅ Complete |
| 6 | Group Policy Update | gpupdate /force | ✅ Complete |

### Key Features Implemented
- ✅ Multiple domain join methods (GUI, PowerShell, Command Line)
- ✅ Comprehensive validation procedures
- ✅ Automated domain join scripts
- ✅ Troubleshooting procedures for common issues
- ✅ Local administrator configuration
- ✅ Network and DNS configuration verification
- ✅ Group Policy application validation

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
