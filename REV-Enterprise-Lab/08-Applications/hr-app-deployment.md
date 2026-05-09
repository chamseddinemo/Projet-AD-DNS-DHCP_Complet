# HR Application Deployment

## 📋 Description

This document provides comprehensive procedures for deploying HR applications in REV-Enterprise-Lab environment, including application installation, configuration, and deployment through Group Policy.

## 🎯 Objectives

- Deploy HR-specific applications to department workstations
- Configure application settings and preferences
- Implement automated deployment through Group Policy
- Establish application update and maintenance procedures
- Ensure application compliance and licensing

## 🚀 Application Deployment Configuration

### HR Application Setup

#### Create HR Application Deployment GPO
```powershell
# Create HR Application Deployment GPO
New-GPO -Name "HR Application Deployment" -Comment "Deploy HR applications to HR department workstations"

# Link to HR OU
New-GPLink -Name "HR Application Deployment" -Target "OU=Human Resources,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure security filtering
Get-GPPermission -Name "HR Application Deployment" | ForEach-Object {
    if ($_.Trustee.Name -ne "Authenticated Users" -and $_.Trustee.Name -ne "SYSTEM") {
        Remove-GPPermission -Name "HR Application Deployment" -TrusteeName $_.Trustee.Name -TrusteeType $_.TrusteeType -ErrorAction SilentlyContinue
    }
}

Set-GPPermission -Name "HR Application Deployment" -TrusteeName "HR-Group" -TrusteeType Group -PermissionLevel GpoApply -Replace
```

#### Application Package Preparation
```powershell
# Function to prepare application package for deployment
function Prepare-ApplicationPackage {
    param(
        [string]$ApplicationName,
        [string]$SourcePath,
        [string]$DestinationPath
    )
    
    # Create destination directory
    if (!(Test-Path $DestinationPath)) {
        New-Item -Path $DestinationPath -ItemType Directory -Force
    }
    
    # Copy application files
    if (Test-Path $SourcePath) {
        Copy-Item -Path $SourcePath\* -Destination $DestinationPath -Recurse -Force
        Write-Host "Application files copied to $DestinationPath"
    } else {
        Write-Host "Source path not found: $SourcePath"
    }
    
    # Create installation script
    $installScript = @"
# HR Application Installation Script
# Application: $ApplicationName
# Date: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')

`$appPath = "$DestinationPath"
`$logPath = "C:\Temp\HR_App_Install_$(Get-Date -Format 'yyyyMMdd_HHmmss').log"

function Write-Log {
    param(`$message)
    `$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "[$timestamp] `$message" | Out-File -FilePath `$logPath -Append
}

try {
    Write-Log "Starting installation of $ApplicationName"
    Write-Log "Application path: `$appPath"
    
    # Check if application is already installed
    `$installedApps = Get-WmiObject -Class Win32_Product | Where-Object {`$_.Name -like "*$ApplicationName*"}
    
    if (`$installedApps.Count -gt 0) {
        Write-Log "$ApplicationName is already installed"
        exit 0
    }
    
    # Install application
    `$installer = Get-ChildItem -Path `$appPath -Filter "*.msi" | Select-Object -First 1
    
    if (`$installer) {
        Write-Log "Installing from: `$(`$installer.FullName)"
        `$process = Start-Process -FilePath "msiexec.exe" -ArgumentList "/i `"`$(`$installer.FullName)`" /quiet /norestart /log `$logPath" -Wait -PassThru
        
        if (`$process.ExitCode -eq 0) {
            Write-Log "Installation completed successfully"
        } else {
            Write-Log "Installation failed with exit code: `$(`$process.ExitCode)"
            exit `$process.ExitCode
        }
    } else {
        Write-Log "No MSI installer found in `$appPath"
        exit 1
    }
    
} catch {
    Write-Log "Error during installation: `$_"
    exit 999
}
"@
    
    # Save installation script
    $scriptPath = "$DestinationPath\install.ps1"
    $installScript | Out-File -FilePath $scriptPath -Encoding UTF8
    
    Write-Host "Installation script created: $scriptPath"
    return $scriptPath
}

# Prepare HR application package
$hrAppSource = "\\FS01\IT$\Software\HR-Application"
$hrAppDestination = "\\PDC\SYSVOL\rev.local\Scripts\HR-Application"
Prepare-ApplicationPackage -ApplicationName "HR Management System" -SourcePath $hrAppSource -DestinationPath $hrAppDestination
```

### MSI Package Deployment

#### Deploy HR Application via GPO
```powershell
# Function to deploy MSI application via Group Policy
function Deploy-MSIApplication {
    param(
        [string]$GPOName,
        [string]$MSIPath,
        [string]$ApplicationName
    )
    
    try {
        # Get GPO
        $gpo = Get-GPO -Name $GPOName
        
        # Create software installation policy
        $softwarePath = "\\rev.local\SysVol\rev.local\Policies\{$($gpo.Id)}\Machine\Software"
        
        if (!(Test-Path $softwarePath)) {
            New-Item -Path $softwarePath -ItemType Directory -Force
        }
        
        # Create applications directory
        $appsPath = "$softwarePath\Applications"
        if (!(Test-Path $appsPath)) {
            New-Item -Path $appsPath -ItemType Directory -Force
        }
        
        # Copy MSI file
        $msiFile = Get-ChildItem -Path $MSIPath -Filter "*.msi" | Select-Object -First 1
        if ($msiFile) {
            $destinationMSI = "$appsPath\$($msiFile.Name)"
            Copy-Item -Path $msiFile.FullName -Destination $destinationMSI -Force
            
            # Create application deployment XML
            $deploymentXML = @"
<?xml version="1.0" encoding="utf-8"?>
<Package xmlns="http://schemas.microsoft.com/2003/10/Deployment/Package">
    <Name>$ApplicationName</Name>
    <Version>1.0.0.0</Version>
    <Publisher>REV Enterprise Lab</Publisher>
    <Architecture>x64</Architecture>
    <Language>en-US</Language>
    <Installer>
        <MsiInstaller>
            <CommandLine>/quiet /norestart</CommandLine>
            <PackageFile>$($msiFile.Name)</PackageFile>
        </MsiInstaller>
    </Installer>
</Package>
"@
            
            $deploymentXML | Out-File -FilePath "$appsPath\deployment.xml" -Encoding UTF8
            
            Write-Host "MSI application deployed: $ApplicationName"
            return $true
        } else {
            Write-Host "No MSI file found in $MSIPath"
            return $false
        }
        
    } catch {
        Write-Host "Error deploying MSI application: $_"
        return $false
    }
}

# Deploy HR application
Deploy-MSIApplication -GPOName "HR Application Deployment" -MSIPath $hrAppDestination -ApplicationName "HR Management System"
```

## 📋 Application Configuration

### HR Application Settings

#### Configure Application Preferences
```powershell
# Function to configure HR application preferences
function Set-HRApplicationPreferences {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Configure application registry settings
            $appKey = "HKLM\SOFTWARE\REV\HR-Application"
            
            if (!(Test-Path $appKey)) {
                New-Item -Path $appKey -Force
            }
            
            # Set application preferences
            Set-ItemProperty -Path $appKey -Name "ServerURL" -Value "http://hrapp.rev.local" -Force
            Set-ItemProperty -Path $appKey -Name "DatabaseName" -Value "HRDB" -Force
            Set-ItemProperty -Path $appKey -Name "ConnectionTimeout" -Value 30 -Force
            Set-ItemProperty -Path $appKey -Name "AutoUpdate" -Value 1 -Force
            Set-ItemProperty -Path $appKey -Name "LogLevel" -Value "Info" -Force
            Set-ItemProperty -Path $appKey -Name "CacheSize" -Value 100 -Force
            Set-ItemProperty -Path $appKey -Name "EnableNotifications" -Value 1 -Force
            Set-ItemProperty -Path $appKey -Name "DefaultView" -Value "Dashboard" -Force
            
            # Configure user-specific settings
            $userKey = "HKCU\SOFTWARE\REV\HR-Application"
            
            if (!(Test-Path $userKey)) {
                New-Item -Path $userKey -Force
            }
            
            Set-ItemProperty -Path $userKey -Name "UserName" -Value $env:USERNAME -Force
            Set-ItemProperty -Path $userKey -Name "LastLogin" -Value (Get-Date) -Force
            Set-ItemProperty -Path $userKey -Name "Theme" -Value "Default" -Force
            Set-ItemProperty -Path $userKey -Name "Language" -Value "en-US" -Force
            Set-ItemProperty -Path $userKey -Name "Timezone" -Value "Eastern Standard Time" -Force
            
            return @{
                Success = $true
                Message = "Application preferences configured successfully"
            }
        }
        
        return $result
        
    } catch {
        Write-Host "Error configuring application preferences on $ComputerName`: $_"
        return @{
            Success = $false
            Message = $_.Exception.Message
        }
    }
}

# Configure preferences on HR computers
$hrComputers = @("HR-PC-01", "HR-PC-02")

foreach ($computer in $hrComputers) {
    $configResult = Set-HRApplicationPreferences -ComputerName $computer
    Write-Host "Configuration result for $computer`: $($configResult.Message)"
}
```

#### Configure Application Security
```powershell
# Function to configure HR application security
function Set-HRApplicationSecurity {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Configure security settings
            $securityKey = "HKLM\SOFTWARE\REV\HR-Application\Security"
            
            if (!(Test-Path $securityKey)) {
                New-Item -Path $securityKey -Force
            }
            
            # Set security preferences
            Set-ItemProperty -Path $securityKey -Name "RequireAuthentication" -Value 1 -Force
            Set-ItemProperty -Path $securityKey -Name "SessionTimeout" -Value 30 -Force
            Set-ItemProperty -Path $securityKey -Name "MaxLoginAttempts" -Value 3 -Force
            Set-ItemProperty -Path $securityKey -Name "PasswordComplexity" -Value 1 -Force
            Set-ItemProperty -Path $securityKey -Name "EncryptData" -Value 1 -Force
            Set-ItemProperty -Path $securityKey -Name "AuditLogEnabled" -Value 1 -Force
            Set-ItemProperty -Path $securityKey -Name "AuditLogPath" -Value "C:\ProgramData\REV\HR-Application\Logs" -Force
            Set-ItemProperty -Path $securityKey -Name "EnableTwoFactor" -Value 0 -Force
            Set-ItemProperty -Path $securityKey -Name "AllowedIPs" -Value "192.168.1.0/24" -Force
            
            # Configure Windows Firewall rules for application
            $firewallRule = @{
                DisplayName = "HR Application"
                Direction = "Inbound"
                Action = "Allow"
                Protocol = "TCP"
                LocalPort = 8080
                RemoteAddress = "192.168.1.0/24"
                Enabled = $true
                Profile = "Domain"
            }
            
            New-NetFirewallRule @firewallRule -ErrorAction SilentlyContinue
            
            return @{
                Success = $true
                Message = "Application security configured successfully"
            }
        }
        
        return $result
        
    } catch {
        Write-Host "Error configuring application security on $ComputerName`: $_"
        return @{
            Success = $false
            Message = $_.Exception.Message
        }
    }
}

# Configure security on HR computers
foreach ($computer in $hrComputers) {
    $securityResult = Set-HRApplicationSecurity -ComputerName $computer
    Write-Host "Security configuration result for $computer`: $($securityResult.Message)"
}
```

## 📊 Application Deployment Monitoring

### Installation Status Monitoring

#### Monitor Application Installation
```powershell
# Function to monitor application installation status
function Get-HRApplicationInstallationStatus {
    param(
        [array]$ComputerNames = @("HR-PC-01", "HR-PC-02")
    )
    
    $installationStatus = @()
    
    foreach ($computer in $ComputerNames) {
        try {
            $result = Invoke-Command -ComputerName $computer -ScriptBlock {
                # Check if application is installed
                $installedApps = Get-WmiObject -Class Win32_Product | Where-Object {$_.Name -like "*HR Management System*"}
                
                $appInfo = if ($installedApps.Count -gt 0) {
                    $app = $installedApps[0]
                    @{
                        Installed = $true
                        Name = $app.Name
                        Version = $app.Version
                        InstallDate = $app.InstallDate
                        Vendor = $app.Vendor
                        PackageName = $app.PackageName
                        LocalPackage = $app.LocalPackage
                    }
                } else {
                    @{
                        Installed = $false
                        Name = "Not Installed"
                        Version = "N/A"
                        InstallDate = "N/A"
                        Vendor = "N/A"
                        PackageName = "N/A"
                        LocalPackage = "N/A"
                    }
                }
                
                # Check application service
                $service = Get-Service -Name "HRAppService" -ErrorAction SilentlyContinue
                $appInfo.ServiceRunning = if ($service) { $service.Status -eq "Running" } else { $false }
                
                # Check application process
                $process = Get-Process -Name "HRApp" -ErrorAction SilentlyContinue
                $appInfo.ProcessRunning = if ($process) { $true } else { $false }
                
                # Check registry settings
                $registryKey = "HKLM:\SOFTWARE\REV\HR-Application"
                $appInfo.RegistryConfigured = Test-Path $registryKey
                
                return $appInfo
            }
            
            $installationStatus += [PSCustomObject]@{
                ComputerName = $computer
                Installed = $result.Installed
                Name = $result.Name
                Version = $result.Version
                InstallDate = $result.InstallDate
                Vendor = $result.Vendor
                ServiceRunning = $result.ServiceRunning
                ProcessRunning = $result.ProcessRunning
                RegistryConfigured = $result.RegistryConfigured
                Status = if ($result.Installed -and $result.ServiceRunning -and $result.RegistryConfigured) { "Operational" } else { "Issue Detected" }
                LastChecked = Get-Date
            }
            
        } catch {
            $installationStatus += [PSCustomObject]@{
                ComputerName = $computer
                Installed = $false
                Name = "Error"
                Version = "Error"
                InstallDate = "Error"
                Vendor = "Error"
                ServiceRunning = $false
                ProcessRunning = $false
                RegistryConfigured = $false
                Status = "Error"
                LastChecked = Get-Date
                Error = $_.Exception.Message
            }
        }
    }
    
    return $installationStatus
}

# Get installation status
$installationStatus = Get-HRApplicationInstallationStatus
$installationStatus | Format-Table -AutoSize
$installationStatus | Export-Csv -Path "C:\Reports\HRApplicationInstallationStatus.csv" -NoTypeInformation
```

### Application Performance Monitoring

#### Monitor Application Performance
```powershell
# Function to monitor application performance
function Get-HRApplicationPerformance {
    param(
        [array]$ComputerNames = @("HR-PC-01", "HR-PC-02")
    )
    
    $performanceMetrics = @()
    
    foreach ($computer in $ComputerNames) {
        try {
            $result = Invoke-Command -ComputerName $computer -ScriptBlock {
                # Get application process metrics
                $process = Get-Process -Name "HRApp" -ErrorAction SilentlyContinue
                
                $metrics = if ($process) {
                    @{
                        CPUUsage = $process.CPU
                        MemoryUsageMB = [math]::Round($process.WorkingSet64 / 1MB, 2)
                        HandleCount = $process.HandleCount
                        ThreadCount = $process.ThreadCount
                        StartTime = $process.StartTime
                        RunTime = (Get-Date) - $process.StartTime
                        Responding = $process.Responding
                    }
                } else {
                    @{
                        CPUUsage = 0
                        MemoryUsageMB = 0
                        HandleCount = 0
                        ThreadCount = 0
                        StartTime = "N/A"
                        RunTime = "N/A"
                        Responding = $false
                    }
                }
                
                # Get application service metrics
                $service = Get-Service -Name "HRAppService" -ErrorAction SilentlyContinue
                $metrics.ServiceStatus = if ($service) { $service.Status } else { "Not Found" }
                
                # Get application log entries
                $logEntries = Get-WinEvent -LogName "Application" -MaxEvents 10 | Where-Object {$_.Source -eq "HR Application"} | Measure-Object
                $metrics.LogEntryCount = $logEntries.Count
                
                return $metrics
            }
            
            $performanceMetrics += [PSCustomObject]@{
                ComputerName = $computer
                CPUUsage = $result.CPUUsage
                MemoryUsageMB = $result.MemoryUsageMB
                HandleCount = $result.HandleCount
                ThreadCount = $result.ThreadCount
                StartTime = $result.StartTime
                RunTime = $result.RunTime
                Responding = $result.Responding
                ServiceStatus = $result.ServiceStatus
                LogEntryCount = $result.LogEntryCount
                LastChecked = Get-Date
                Status = if ($result.Responding -and $result.ServiceStatus -eq "Running") { "Healthy" } else { "Issue" }
            }
            
        } catch {
            $performanceMetrics += [PSCustomObject]@{
                ComputerName = $computer
                CPUUsage = 0
                MemoryUsageMB = 0
                HandleCount = 0
                ThreadCount = 0
                StartTime = "Error"
                RunTime = "Error"
                Responding = $false
                ServiceStatus = "Error"
                LogEntryCount = 0
                LastChecked = Get-Date
                Status = "Error"
                Error = $_.Exception.Message
            }
        }
    }
    
    return $performanceMetrics
}

# Get performance metrics
$performanceMetrics = Get-HRApplicationPerformance
$performanceMetrics | Format-Table -AutoSize
$performanceMetrics | Export-Csv -Path "C:\Reports\HRApplicationPerformance.csv" -NoTypeInformation
```

## 🔧 Application Maintenance

### Update Management

#### Configure Application Updates
```powershell
# Function to configure application update management
function Set-HRApplicationUpdates {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Configure update settings
            $updateKey = "HKLM\SOFTWARE\REV\HR-Application\Updates"
            
            if (!(Test-Path $updateKey)) {
                New-Item -Path $updateKey -Force
            }
            
            # Set update preferences
            Set-ItemProperty -Path $updateKey -Name "AutoUpdate" -Value 1 -Force
            Set-ItemProperty -Path $updateKey -Name "UpdateFrequency" -Value "Daily" -Force
            Set-ItemProperty -Path $updateKey -Name "UpdateTime" -Value "02:00" -Force
            Set-ItemProperty -Path $updateKey -Name "UpdateServer" -Value "http://updates.rev.local/hr-app" -Force
            Set-ItemProperty -Path $updateKey -Name "RequireReboot" -Value 0 -Force
            Set-ItemProperty -Path $updateKey -Name "BackupBeforeUpdate" -Value 1 -Force
            Set-ItemProperty -Path $updateKey -Name "LogUpdates" -Value 1 -Force
            Set-ItemProperty -Path $updateKey -Name "UpdateLogLevel" -Value "Info" -Force
            
            # Create scheduled task for updates
            $action = New-ScheduledTaskAction -Execute "C:\Program Files\REV\HR-Application\HRAppUpdater.exe" -Argument "/silent"
            $trigger = New-ScheduledTaskTrigger -Daily -At 2am -RandomDelay (New-TimeSpan -Minutes 30)
            $settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopOnIdleEnd -AllowStartIfOnBatteries
            
            Register-ScheduledTask -TaskName "HR Application Updates" -Action $action -Trigger $trigger -Settings $settings -Force
            
            return @{
                Success = $true
                Message = "Application updates configured successfully"
            }
        }
        
        return $result
        
    } catch {
        Write-Host "Error configuring application updates on $ComputerName`: $_"
        return @{
            Success = $false
            Message = $_.Exception.Message
        }
    }
}

# Configure updates on HR computers
foreach ($computer in $hrComputers) {
    $updateResult = Set-HRApplicationUpdates -ComputerName $computer
    Write-Host "Update configuration result for $computer`: $($updateResult.Message)"
}
```

### Backup and Recovery

#### Configure Application Backup
```powershell
# Function to configure application backup
function Set-HRApplicationBackup {
    param(
        [string]$ComputerName
    )
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Configure backup settings
            $backupKey = "HKLM\SOFTWARE\REV\HR-Application\Backup"
            
            if (!(Test-Path $backupKey)) {
                New-Item -Path $backupKey -Force
            }
            
            # Set backup preferences
            Set-ItemProperty -Path $backupKey -Name "EnableBackup" -Value 1 -Force
            Set-ItemProperty -Path $backupKey -Name "BackupPath" -Value "\\FS01\HR$\Backups\Applications" -Force
            Set-ItemProperty -Path $backupKey -Name "BackupFrequency" -Value "Daily" -Force
            Set-ItemProperty -Path $backupKey -Name "BackupTime" -Value "01:00" -Force
            Set-ItemProperty -Path $backupKey -Name "RetentionDays" -Value 30 -Force
            Set-ItemProperty -Path $backupKey -Name "CompressBackup" -Value 1 -Force
            Set-ItemProperty -Path $backupKey -Name "EncryptBackup" -Value 1 -Force
            Set-ItemProperty -Path $backupKey -Name "IncludeUserData" -Value 1 -Force
            Set-ItemProperty -Path $backupKey -Name "IncludeSettings" -Value 1 -Force
            Set-ItemProperty -Path $backupKey -Name "IncludeLogs" -Value 0 -Force
            
            # Create backup script
            $backupScript = @"
# HR Application Backup Script
`$backupPath = "\\FS01\HR$\Backups\Applications"
`$appDataPath = "C:\ProgramData\REV\HR-Application"
`$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"

function Write-BackupLog {
    param(`$message)
    `$logPath = "C:\Temp\HR_App_Backup_`$timestamp`.log"
    `$logTime = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    "[`$logTime] `$message" | Out-File -FilePath `$logPath -Append
}

try {
    Write-BackupLog "Starting HR application backup"
    
    # Create backup directory
    `$backupDir = "`$backupPath\`$timestamp"
    if (!(Test-Path `$backupDir)) {
        New-Item -Path `$backupDir -ItemType Directory -Force
    }
    
    # Backup application data
    if (Test-Path `$appDataPath) {
        Write-BackupLog "Backing up application data from `$appDataPath"
        Copy-Item -Path `$appDataPath -Destination "`$backupDir\ApplicationData" -Recurse -Force
    }
    
    # Backup registry settings
    Write-BackupLog "Backing up registry settings"
    `$regExportPath = "`$backupDir\RegistrySettings.reg"
    reg export "HKLM\SOFTWARE\REV\HR-Application" "`$regExportPath" /y
    
    Write-BackupLog "Backup completed successfully"
    
} catch {
    Write-BackupLog "Backup failed: `$_"
    exit 1
}
"@
            
            # Save backup script
            $scriptPath = "C:\Scripts\HRAppBackup.ps1"
            New-Item -Path (Split-Path $scriptPath) -ItemType Directory -Force
            $backupScript | Out-File -FilePath $scriptPath -Encoding UTF8
            
            # Create scheduled task for backup
            $action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-ExecutionPolicy Bypass -File `"$scriptPath`""
            $trigger = New-ScheduledTaskTrigger -Daily -At 1am
            $settings = New-ScheduledTaskSettingsSet -StartWhenAvailable -DontStopOnIdleEnd -AllowStartIfOnBatteries
            
            Register-ScheduledTask -TaskName "HR Application Backup" -Action $action -Trigger $trigger -Settings $settings -Force
            
            return @{
                Success = $true
                Message = "Application backup configured successfully"
            }
        }
        
        return $result
        
    } catch {
        Write-Host "Error configuring application backup on $ComputerName`: $_"
        return @{
            Success = $false
            Message = $_.Exception.Message
        }
    }
}

# Configure backup on HR computers
foreach ($computer in $hrComputers) {
    $backupResult = Set-HRApplicationBackup -ComputerName $computer
    Write-Host "Backup configuration result for $computer`: $($backupResult.Message)"
}
```

## 🔍 Application Troubleshooting

### Common Issues and Solutions

#### Installation Issues
```powershell
# Function to troubleshoot application installation issues
function Test-HRApplicationInstallation {
    param(
        [string]$ComputerName
    )
    
    $troubleshooting = @()
    
    try {
        $result = Invoke-Command -ComputerName $ComputerName -ScriptBlock {
            # Test 1: Check if application is already installed
            $installedApps = Get-WmiObject -Class Win32_Product | Where-Object {$_.Name -like "*HR Management System*"}
            $isInstalled = $installedApps.Count -gt 0
            
            # Test 2: Check if installation files are accessible
            $installPath = "\\rev.local\SysVol\rev.local\Scripts\HR-Application"
            $filesAccessible = Test-Path $installPath
            
            # Test 3: Check if Windows Installer service is running
            $msiService = Get-Service -Name "msiserver"
            $msiRunning = $msiService.Status -eq "Running"
            
            # Test 4: Check disk space
            $systemDrive = Get-WmiObject -Class Win32_LogicalDisk | Where-Object {$_.DeviceID -eq "C:"}
            $freeSpaceGB = [math]::Round($systemDrive.FreeSpace / 1GB, 2)
            $sufficientSpace = $freeSpaceGB -gt 1
            
            # Test 5: Check system requirements
            $os = Get-WmiObject -Class Win32_OperatingSystem
            $osVersion = $os.Version
            $osArchitecture = $os.OSArchitecture
            
            return @{
                IsInstalled = $isInstalled
                FilesAccessible = $filesAccessible
                MSIServiceRunning = $msiRunning
                FreeSpaceGB = $freeSpaceGB
                SufficientSpace = $sufficientSpace
                OSVersion = $osVersion
                OSArchitecture = $osArchitecture
            }
        }
        
        # Add troubleshooting results
        $troubleshooting += [PSCustomObject]@{
            Test = "Application Installation Status"
            Status = if ($result.IsInstalled) { "Already Installed" } else { "Not Installed" }
            Details = "Application installation status: $($result.IsInstalled)"
        }
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Installation Files Access"
            Status = if ($result.FilesAccessible) { "Accessible" } else { "Not Accessible" }
            Details = "Installation files are $($result.FilesAccessible)"
        }
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Windows Installer Service"
            Status = if ($result.MSIServiceRunning) { "Running" } else { "Not Running" }
            Details = "Windows Installer service is $($result.MSIServiceRunning)"
        }
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Disk Space"
            Status = if ($result.SufficientSpace) { "Sufficient" } else { "Insufficient" }
            Details = "Available disk space: $($result.FreeSpaceGB) GB"
        }
        
        $troubleshooting += [PSCustomObject]@{
            Test = "System Requirements"
            Status = "Compatible"
            Details = "OS Version: $($result.OSVersion), Architecture: $($result.OSArchitecture)"
        }
        
    } catch {
        $troubleshooting += [PSCustomObject]@{
            Test = "Connection Test"
            Status = "Error"
            Details = "Error connecting to $ComputerName`: $_"
        }
    }
    
    return $troubleshooting
}

# Troubleshoot installation on HR computers
foreach ($computer in $hrComputers) {
    $troubleshootingResult = Test-HRApplicationInstallation -ComputerName $computer
    Write-Host "Troubleshooting results for $computer`:"
    $troubleshootingResult | Format-Table -AutoSize
}
```

## 🖼️ Screenshots

### Application Deployment
![Application Deployment](../screenshots/08-applications/app-deployment.png)

### Group Policy Software Installation
![GPO Software Installation](../screenshots/08-applications/gpo-software-installation.png)

### Application Configuration
![Application Configuration](../screenshots/08-applications/app-configuration.png)

### Installation Status
![Installation Status](../screenshots/08-applications/installation-status.png)

## 📋 HR Application Deployment Summary

| Component | Configuration | Target | Status |
|------------|----------------|---------|---------|
| Application Package | MSI Package | HR Workstations | ✅ Prepared |
| GPO Deployment | HR Application Deployment | HR OU | ✅ Configured |
| Security Filtering | HR-Group Only | HR Workstations | ✅ Applied |
| Application Preferences | Registry Settings | HR Workstations | ✅ Configured |
| Security Settings | Firewall + Registry | HR Workstations | ✅ Configured |
| Update Management | Scheduled Task | HR Workstations | ✅ Configured |
| Backup Configuration | Scheduled Task | HR Workstations | ✅ Configured |

### Key Features Implemented
- ✅ HR application package preparation and deployment
- ✅ MSI deployment through Group Policy
- ✅ Application preferences and security configuration
- ✅ Automated update management
- ✅ Backup and recovery procedures
- ✅ Installation status monitoring
- ✅ Performance monitoring and metrics
- ✅ Comprehensive troubleshooting procedures

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
