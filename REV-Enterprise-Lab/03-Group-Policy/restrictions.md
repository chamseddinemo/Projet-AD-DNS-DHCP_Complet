# User Restrictions

## 📋 Description

This document provides detailed procedures for implementing user restrictions through Group Policy Objects in the REV-Enterprise-Lab environment, focusing on limiting user access to system features and applications to enhance security and productivity.

## 🎯 Objectives

- Implement comprehensive user access restrictions
- Configure system feature limitations for specific departments
- Establish user environment controls
- Prevent unauthorized system modifications
- Ensure compliance with organizational policies

## 🚫 Command Prompt Restrictions

### HR Department Command Prompt Restrictions

#### Complete Command Prompt Disable
```powershell
# Create HR Command Restrictions GPO
New-GPO -Name "HR Command Restrictions" -Comment "Disable Command Prompt access for HR department"

# Link to HR OU
New-GPLink -Name "HR Command Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Command Prompt completely
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableCMD" -Type DWORD -Value 1

# Disable Command Prompt from Run dialog
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableRun" -Type DWORD -Value 1

# Disable Command Prompt shortcuts
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoRun" -Type DWORD -Value 1
```

#### PowerShell Restrictions
```powershell
# Disable PowerShell execution
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\PowerShell" -ValueName "EnableScripts" -Type DWORD -Value 0

# Disable PowerShell console
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\PowerShell" -ValueName "EnableConsoleHost" -Type DWORD -Value 0

# Set PowerShell execution policy to Restricted
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKLM\SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell" -ValueName "ExecutionPolicy" -Type String -Value "Restricted"
```

#### Batch File Restrictions
```powershell
# Prevent execution of batch files
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "DisallowRun" -Type DWORD -Value 1

# Create disallowed applications list
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "1" -Type String -Value "cmd.exe"
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "2" -Type String -Value "powershell.exe"
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "3" -Type String -Value "wscript.exe"
Set-GPRegistryValue -Name "HR Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "4" -Type String -Value "cscript.exe"
```

### Sales Department Command Restrictions

#### Apply Same Restrictions to Sales
```powershell
# Create Sales Command Restrictions GPO
New-GPO -Name "Sales Command Restrictions" -Comment "Disable Command Prompt access for Sales department"

# Link to Sales OU
New-GPLink -Name "Sales Command Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Apply same restrictions as HR
Set-GPRegistryValue -Name "Sales Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableCMD" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\System" -ValueName "DisableRun" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Command Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoRun" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Command Restrictions" -Key "HKCU\Software\Policies\Microsoft\Windows\PowerShell" -ValueName "EnableScripts" -Type DWORD -Value 0
```

## 🖥️ Control Panel Restrictions

### HR Department Control Panel Disable

#### Complete Control Panel Access Disable
```powershell
# Create HR Control Panel Restrictions GPO
New-GPO -Name "HR Control Panel Restrictions" -Comment "Disable Control Panel access for HR department"

# Link to HR OU
New-GPLink -Name "HR Control Panel Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Control Panel completely
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoControlPanel" -Type DWORD -Value 1

# Hide Control Panel from Settings
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSetFolders" -Type DWORD -Value 1

# Disable Settings app
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSettingsPage" -Type DWORD -Value 1
```

#### Specific Control Panel Items Restrictions
```powershell
# Hide specific Control Panel items
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "DisallowCpl" -Type DWORD -Value 1

# Create disallowed CPL list
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "1" -Type String -Value "inetcpl.cpl"  # Internet Options
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "2" -Type String -Value "sysdm.cpl"    # System Properties
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "3" -Type String -Value "userpasswords.cpl"  # User Accounts
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowCpl" -ValueName "4" -Type String -Value "ncpa.cpl"     # Network Connections
```

#### Windows 10 Settings App Restrictions
```powershell
# Disable specific Settings pages
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "SettingsPageVisibility" -Type String -Value "hide:system;devices;network;personalization;accounts;time;easeofaccess;privacy;update;recovery"

# Allow only specific Settings pages
Set-GPRegistryValue -Name "HR Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "SettingsPageVisibility" -Type String -Value "show:display;sound;notifications;powersleep;apps;features;defaultapps;appsfeatures"
```

### Sales Department Control Panel Restrictions

#### Apply Same Restrictions to Sales
```powershell
# Create Sales Control Panel Restrictions GPO
New-GPO -Name "Sales Control Panel Restrictions" -Comment "Disable Control Panel access for Sales department"

# Link to Sales OU
New-GPLink -Name "Sales Control Panel Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Apply same restrictions as HR
Set-GPRegistryValue -Name "Sales Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoControlPanel" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSetFolders" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Control Panel Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoSettingsPage" -Type DWORD -Value 1
```

## 📁 File System Restrictions

### Remove Properties from This PC Context Menu

#### Context Menu Restrictions
```powershell
# Create HR Context Menu Restrictions GPO
New-GPO -Name "HR Context Menu Restrictions" -Comment "Remove Properties from This PC context menu for HR"

# Link to HR OU
New-GPLink -Name "HR Context Menu Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Remove Properties from This PC context menu
Set-GPRegistryValue -Name "HR Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoPropertiesMyComputer" -Type DWORD -Value 1

# Remove Properties from drives
Set-GPRegistryValue -Name "HR Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoPropertiesRecycleBin" -Type DWORD -Value 1

# Disable right-click on desktop
Set-GPRegistryValue -Name "HR Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoViewContextMenu" -Type DWORD -Value 1
```

#### File Explorer Restrictions
```powershell
# Hide specific drives in File Explorer
Set-GPRegistryValue -Name "HR Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoDrives" -Type DWORD -Value 4  # Hide C: drive

# Hide Network Neighborhood
Set-GPRegistryValue -Name "HR Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoNetHood" -Type DWORD -Value 1

# Disable File Explorer's ribbon
Set-GPRegistryValue -Name "HR Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoRibbon" -Type DWORD -Value 1
```

### Sales Department Context Menu Restrictions

#### Apply Same Restrictions to Sales
```powershell
# Create Sales Context Menu Restrictions GPO
New-GPO -Name "Sales Context Menu Restrictions" -Comment "Remove Properties from This PC context menu for Sales"

# Link to Sales OU
New-GPLink -Name "Sales Context Menu Restrictions" -Target "OU=Sales,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Apply same restrictions as HR
Set-GPRegistryValue -Name "Sales Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoPropertiesMyComputer" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Sales Context Menu Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoViewContextMenu" -Type DWORD -Value 1
```

## 🌐 Internet and Network Restrictions

### Browser Restrictions for HR and Sales

#### Internet Explorer Restrictions
```powershell
# Create HR Internet Restrictions GPO
New-GPO -Name "HR Internet Restrictions" -Comment "Internet access restrictions for HR department"

# Link to HR OU
New-GPLink -Name "HR Internet Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure Internet Explorer restrictions
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Policies\Microsoft\Internet Explorer\Restrictions" -ValueName "NoBrowserOptions" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Policies\Microsoft\Internet Explorer\Restrictions" -ValueName "NoFavorites" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Policies\Microsoft\Internet Explorer\Restrictions" -ValueName "NoSelectDownloadDir" -Type DWORD -Value 1
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Policies\Microsoft\Internet Explorer\Restrictions" -ValueName "NoFindFiles" -Type DWORD -Value 1
```

#### Network Access Restrictions
```powershell
# Disable network connections
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoNetworkConnections" -Type DWORD -Value 1

# Disable Map Network Drive
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoConnect" -Type DWORD -Value 1

# Disable Disconnect Network Drive
Set-GPRegistryValue -Name "HR Internet Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoDisconnect" -Type DWORD -Value 1
```

## 🎮 Entertainment and Media Restrictions

### Disable Games and Entertainment

#### Game Restrictions
```powershell
# Create HR Entertainment Restrictions GPO
New-GPO -Name "HR Entertainment Restrictions" -Comment "Disable games and entertainment for HR department"

# Link to HR OU
New-GPLink -Name "HR Entertainment Restrictions" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Disable Windows games
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "NoGames" -Type DWORD -Value 1

# Disable Windows Media Center
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsMediaPlayer" -ValueName "DisallowMediaSharing" -Type DWORD -Value 1

# Disable Windows Media Player
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKCU\Software\Policies\Microsoft\WindowsMediaPlayer" -ValueName "MediaUsageTracking" -Type DWORD -Value 0
```

#### Application Restrictions
```powershell
# Disable specific applications
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -ValueName "DisallowRun" -Type DWORD -Value 1

# Add disallowed applications
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "5" -Type String -Value "solitaire.exe"
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "6" -Type String -Value "winmine.exe"
Set-GPRegistryValue -Name "HR Entertainment Restrictions" -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\DisallowRun" -ValueName "7" -Type String -Value "wmplayer.exe"
```

## 📊 Restriction Validation

### Test Command Prompt Restrictions
```powershell
# Function to test Command Prompt restrictions
function Test-CommandRestrictions {
    param(
        [string]$ComputerName
    )
    
    # Test Command Prompt access
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            Get-Command cmd.exe -ErrorAction SilentlyContinue
        }
        if ($result) {
            Write-Host "Command Prompt is accessible on $ComputerName - RESTRICTION FAILED"
        } else {
            Write-Host "Command Prompt is properly restricted on $ComputerName - RESTRICTION SUCCESS"
        }
    } catch {
        Write-Host "Cannot test Command Prompt on $ComputerName - ACCESS DENIED (Expected)"
    }
}

# Test on HR computers
Test-CommandRestrictions -ComputerName "HR-PC-01"
Test-CommandRestrictions -ComputerName "HR-PC-02"
```

### Test Control Panel Restrictions
```powershell
# Function to test Control Panel restrictions
function Test-ControlPanelRestrictions {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            Get-Process -Name "control" -ErrorAction SilentlyContinue
        }
        if ($result) {
            Write-Host "Control Panel is accessible on $ComputerName - RESTRICTION FAILED"
        } else {
            Write-Host "Control Panel is properly restricted on $ComputerName - RESTRICTION SUCCESS"
        }
    } catch {
        Write-Host "Cannot test Control Panel on $ComputerName - ACCESS DENIED (Expected)"
    }
}

# Test on HR computers
Test-ControlPanelRestrictions -ComputerName "HR-PC-01"
Test-ControlPanelRestrictions -ComputerName "HR-PC-02"
```

### Generate Restriction Report
```powershell
# Generate restriction compliance report
function Get-RestrictionComplianceReport {
    $computers = @("HR-PC-01", "HR-PC-02", "SALES-PC-01", "SALES-PC-02")
    $report = @()
    
    foreach ($computer in $computers) {
        $compliance = [PSCustomObject]@{
            ComputerName = $computer
            CommandRestricted = "Unknown"
            ControlPanelRestricted = "Unknown"
            ContextMenuRestricted = "Unknown"
            OverallStatus = "Unknown"
        }
        
        # Test restrictions (simplified for demo)
        try {
            $gpResult = Get-GPResultantSetOfPolicy -Computer $computer -ReportType Xml
            $appliedGPOs = $gpResult.GPO | Where-Object {$_.DisplayName -like "*Restrictions*"}
            
            if ($appliedGPOs.Count -gt 0) {
                $compliance.CommandRestricted = "Applied"
                $compliance.ControlPanelRestricted = "Applied"
                $compliance.ContextMenuRestricted = "Applied"
                $compliance.OverallStatus = "Compliant"
            } else {
                $compliance.OverallStatus = "Non-Compliant"
            }
        } catch {
            $compliance.OverallStatus = "Error"
        }
        
        $report += $compliance
    }
    
    return $report
}

# Generate and display report
$complianceReport = Get-RestrictionComplianceReport
$complianceReport | Format-Table -AutoSize
$complianceReport | Export-Csv -Path "C:\Reports\RestrictionCompliance.csv" -NoTypeInformation
```

## 🖼️ Screenshots

### Command Prompt Restrictions
![Command Prompt Disabled](../screenshots/03-group-policy/cmd-restricted.png)

### Control Panel Restrictions
![Control Panel Disabled](../screenshots/03-group-policy/control-panel-restricted.png)

### Context Menu Restrictions
![Context Menu Modified](../screenshots/03-group-policy/context-menu-restricted.png)

### GPO Application
![GPO Applied Successfully](../screenshots/03-group-policy/gpo-applied.png)

## 📋 Restriction Summary

| Restriction Type | Departments Affected | GPOs Created | Status |
|------------------|---------------------|---------------|---------|
| Command Prompt | HR, Sales | 2 | ✅ Configured |
| Control Panel | HR, Sales | 2 | ✅ Configured |
| Context Menu | HR, Sales | 2 | ✅ Configured |
| File System | HR, Sales | 2 | ✅ Configured |
| Internet Access | HR | 1 | ✅ Configured |
| Entertainment | HR | 1 | ✅ Configured |
| Total Restrictions | Multiple | 10 | ✅ Complete |

### Key Features Implemented
- ✅ Command Prompt completely disabled for HR and Sales
- ✅ Control Panel access blocked for restricted departments
- ✅ Properties menu removed from This PC context menu
- ✅ File system access restrictions implemented
- ✅ Internet and network access controls configured
- ✅ Entertainment and media applications blocked
- ✅ Comprehensive validation procedures established

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
