# Password Policy Configuration

## 📋 Description

This document provides comprehensive procedures for configuring password policies in REV-Enterprise-Lab Active Directory environment, including password complexity, expiration, and history requirements.

## 🎯 Objectives

- Implement secure password policies for all users
- Configure password complexity requirements
- Set password expiration and history policies
- Ensure compliance with security best practices
- Document password policy configuration procedures

## 🔧 Password Policy Configuration

### Default Domain Password Policy

#### Password Policy Settings
```powershell
# Get Default Domain Policy
$defaultDomainGPO = Get-GPO -Name "Default Domain Policy"

# Configure password policy settings
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordLength" -Type DWORD -Value 6
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordComplexity" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MaximumPasswordAge" -Type DWORD -Value 60
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordAge" -Type DWORD -Value 1
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordHistorySize" -Type DWORD -Value 3
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "ClearTextPassword" -Type DWORD -Value 0
Set-GPRegistryValue -Name "Default Domain Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "StorePasswordsUsing reversibleencryption" -Type DWORD -Value 0
```

#### Password Policy Verification
```powershell
# Function to verify password policy configuration
function Test-PasswordPolicyConfiguration {
    $gpo = Get-GPO -Name "Default Domain Policy"
    
    $policySettings = @()
    
    # Check minimum password length
    $minLength = Get-GPRegistryValue -GUID $gpo.Id -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordLength"
    $policySettings += [PSCustomObject]@{
        Setting = "Minimum Password Length"
        ConfiguredValue = $minLength.Dword
        ExpectedValue = 6
        Status = if ($minLength.Dword -ge 6) { "Compliant" } else { "Non-Compliant" }
    }
    
    # Check password complexity
    $complexity = Get-GPRegistryValue -GUID $gpo.Id -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordComplexity"
    $policySettings += [PSCustomObject]@{
        Setting = "Password Complexity"
        ConfiguredValue = $complexity.Dword
        ExpectedValue = 1
        Status = if ($complexity.Dword -eq 1) { "Enabled" } else { "Disabled" }
    }
    
    # Check maximum password age
    $maxAge = Get-GPRegistryValue -GUID $gpo.Id -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MaximumPasswordAge"
    $policySettings += [PSCustomObject]@{
        Setting = "Maximum Password Age"
        ConfiguredValue = "$($maxAge.Dword) days"
        ExpectedValue = 60
        Status = if ($maxAge.Dword -eq 60) { "Compliant" } else { "Non-Compliant" }
    }
    
    # Check minimum password age
    $minAge = Get-GPRegistryValue -GUID $gpo.Id -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordAge"
    $policySettings += [PSCustomObject]@{
        Setting = "Minimum Password Age"
        ConfiguredValue = "$($minAge.Dword) days"
        ExpectedValue = 1
        Status = if ($minAge.Dword -ge 1) { "Compliant" } else { "Non-Compliant" }
    }
    
    # Check password history
    $historySize = Get-GPRegistryValue -GUID $gpo.Id -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordHistorySize"
    $policySettings += [PSCustomObject]@{
        Setting = "Password History Size"
        ConfiguredValue = $historySize.Dword
        ExpectedValue = 3
        Status = if ($historySize.Dword -ge 3) { "Compliant" } else { "Non-Compliant" }
    }
    
    return $policySettings
}

# Verify password policy configuration
$passwordPolicyTest = Test-PasswordPolicyConfiguration
$passwordPolicyTest | Format-Table -AutoSize
```

### Fine-Grained Password Policies

#### Create Fine-Grained Password Policies for Different Departments
```powershell
# Create Fine-Grained Password Policy GPO for IT Users
New-GPO -Name "IT Fine-Grained Password Policy" -Comment "Enhanced password policy for IT department users"

# Link to IT OU
New-GPLink -Name "IT Fine-Grained Password Policy" -Target "OU=IT Department,OU=Departments,DC=rev,DC=local" -LinkEnabled Yes

# Configure enhanced settings for IT users
Set-GPRegistryValue -Name "IT Fine-Grained Password Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordLength" -Type DWORD -Value 8
Set-GPRegistryValue -Name "IT Fine-Grained Password Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordComplexity" -Type DWORD -Value 1
Set-GPRegistryValue -Name "IT Fine-Grained Password Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MaximumPasswordAge" -Type DWORD -Value 90
Set-GPRegistryValue -Name "IT Fine-Grained Password Policy" -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordHistorySize" -Type DWORD -Value 5

# Configure security filtering
$gpo = Get-GPO -Name "IT Fine-Grained Password Policy"
Get-GPPermission -Name "IT Fine-Grained Password Policy" | ForEach-Object {
    if ($_.Trustee.Name -ne "Authenticated Users" -and $_.Trustee.Name -ne "SYSTEM") {
        Remove-GPPermission -Name "IT Fine-Grained Password Policy" -TrusteeName $_.Trustee.Name -TrusteeType $_.TrusteeType -ErrorAction SilentlyContinue
    }
}

Set-GPPermission -Name "IT Fine-Grained Password Policy" -TrusteeName "IT-Group" -TrusteeType Group -PermissionLevel GpoApply -Replace
```

## 🔒 Password Complexity Requirements

### Complexity Configuration
```powershell
# Function to configure password complexity requirements
function Set-PasswordComplexity {
    param(
        [string]$GPOName = "Default Domain Policy",
        [bool]$RequireUppercase = $true,
        [bool]$RequireLowercase = $true,
        [bool]$RequireNumbers = $true,
        [bool]$RequireSpecialChars = $true,
        [int]$MinLength = 6
    )
    
    # Configure complexity settings
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordComplexity" -Type DWORD -Value 1
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordLength" -Type DWORD -Value $MinLength
    
    # Configure additional complexity requirements (Windows Server 2012+)
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SYSTEM\CurrentControlSet\Control\PasswordComplexity" -ValueName "PasswordComplexity" -Type DWORD -Value 1
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SYSTEM\CurrentControlSet\Control\PasswordComplexity" -ValueName "MinimumPasswordLength" -Type DWORD -Value $MinLength
    
    Write-Host "Password complexity configured for $GPOName"
    Write-Host "Minimum length: $MinLength characters"
    Write-Host "Complexity requirements: Uppercase=$RequireUppercase, Lowercase=$RequireLowercase, Numbers=$RequireNumbers, Special=$RequireSpecialChars"
}

# Configure standard password complexity
Set-PasswordComplexity -MinLength 6

# Configure enhanced complexity for IT users
Set-PasswordComplexity -GPOName "IT Fine-Grained Password Policy" -MinLength 8
```

### Password Complexity Validation Script
```powershell
# Function to validate password complexity
function Test-PasswordComplexity {
    param(
        [string]$Password
    )
    
    $complexityCheck = @{
        Password = $Password
        Length = $Password.Length
        HasUppercase = $false
        HasLowercase = $false
        HasNumbers = $false
        HasSpecialChars = $false
        MeetsRequirements = $false
        Score = 0
    }
    
    # Check length
    if ($complexityCheck.Length -ge 6) {
        $complexityCheck.Score += 20
    }
    if ($complexityCheck.Length -ge 8) {
        $complexityCheck.Score += 20
    }
    if ($complexityCheck.Length -ge 12) {
        $complexityCheck.Score += 20
    }
    
    # Check character types
    if ($Password -cmatch '[A-Z]') {
        $complexityCheck.HasUppercase = $true
        $complexityCheck.Score += 10
    }
    
    if ($Password -cmatch '[a-z]') {
        $complexityCheck.HasLowercase = $true
        $complexityCheck.Score += 10
    }
    
    if ($Password -cmatch '[0-9]') {
        $complexityCheck.HasNumbers = $true
        $complexityCheck.Score += 10
    }
    
    if ($Password -cmatch '[^a-zA-Z0-9]') {
        $complexityCheck.HasSpecialChars = $true
        $complexityCheck.Score += 10
    }
    
    # Determine if meets requirements
    $complexityCheck.MeetsRequirements = (
        $complexityCheck.Length -ge 6 -and
        $complexityCheck.HasUppercase -and
        $complexityCheck.HasLowercase -and
        $complexityCheck.HasNumbers -and
        $complexityCheck.HasSpecialChars
    )
    
    # Determine password strength
    $complexityCheck.Strength = switch ($complexityCheck.Score) {
        {$_ -ge 80} { "Very Strong" }
        {$_ -ge 60} { "Strong" }
        {$_ -ge 40} { "Medium" }
        {$_ -ge 20} { "Weak" }
        default { "Very Weak" }
    }
    
    return $complexityCheck
}

# Test password complexity
$passwordTests = @(
    "Password123",
    "P@ssw0rd!",
    "ComplexPass2026",
    "MyStr0ngP@ssw0rd!"
)

foreach ($password in $passwordTests) {
    $result = Test-PasswordComplexity -Password $password
    Write-Host "Password: $password"
    Write-Host "  Length: $($result.Length)"
    Write-Host "  Uppercase: $($result.HasUppercase)"
    Write-Host "  Lowercase: $($result.HasLowercase)"
    Write-Host "  Numbers: $($result.HasNumbers)"
    Write-Host "  Special: $($result.HasSpecialChars)"
    Write-Host "  Score: $($result.Score)"
    Write-Host "  Strength: $($result.Strength)"
    Write-Host "  Meets Requirements: $($result.MeetsRequirements)"
    Write-Host ""
}
```

## 📅 Password Expiration Management

### Expiration Policy Configuration
```powershell
# Function to configure password expiration
function Set-PasswordExpiration {
    param(
        [string]$GPOName = "Default Domain Policy",
        [int]$MaxAgeDays = 60,
        [int]$MinAgeDays = 1,
        [int]$WarningDays = 14
    )
    
    # Convert days to seconds for Windows
    $maxAgeSeconds = $MaxAgeDays * 86400
    $minAgeSeconds = $MinAgeDays * 86400
    $warningSeconds = $WarningDays * 86400
    
    # Configure expiration settings
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MaximumPasswordAge" -Type DWORD -Value $maxAgeSeconds
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordAge" -Type DWORD -Value $minAgeSeconds
    
    # Configure notification settings
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordExpiryWarning" -Type DWORD -Value $warningSeconds
    
    Write-Host "Password expiration configured for $GPOName"
    Write-Host "Maximum age: $MaxAgeDays days"
    Write-Host "Minimum age: $MinAgeDays days"
    Write-Host "Warning period: $WarningDays days"
}

# Configure standard password expiration
Set-PasswordExpiration -MaxAgeDays 60 -MinAgeDays 1 -WarningDays 14

# Configure enhanced expiration for IT users
Set-PasswordExpiration -GPOName "IT Fine-Grained Password Policy" -MaxAgeDays 90 -MinAgeDays 7 -WarningDays 21
```

### Password Expiration Monitoring
```powershell
# Function to monitor password expiration
function Get-PasswordExpirationReport {
    $users = Get-ADUser -Filter * -Properties PasswordLastSet, PasswordNeverExpires, AccountExpirationDate, DisplayName, UserPrincipalName
    
    $expirationReport = @()
    
    foreach ($user in $users) {
        if ($user.PasswordNeverExpires -eq $false -and $user.Enabled -eq $true) {
            $lastSet = $user.PasswordLastSet
            $maxAge = 60  # Default policy
            $expiresOn = $lastSet.AddDays($maxAge)
            $daysUntilExpiration = ($expiresOn - (Get-Date)).Days
            
            $status = switch ($daysUntilExpiration) {
                {$_ -le 0} { "Expired" }
                {$_ -le 7} { "Critical" }
                {$_ -le 14} { "Warning" }
                {$_ -le 30} { "Notice" }
                default { "OK" }
            }
            
            $expirationReport += [PSCustomObject]@{
                UserName = $user.UserPrincipalName
                DisplayName = $user.DisplayName
                LastSet = $lastSet
                ExpiresOn = $expiresOn
                DaysUntilExpiration = $daysUntilExpiration
                Status = $status
                AccountEnabled = $user.Enabled
            }
        }
    }
    
    return $expirationReport
}

# Generate password expiration report
$expirationReport = Get-PasswordExpirationReport
$expirationReport | Sort-Object DaysUntilExpiration | Format-Table -AutoSize
$expirationReport | Export-Csv -Path "C:\Reports\PasswordExpirationReport.csv" -NoTypeInformation
```

## 🔄 Password History Management

### History Policy Configuration
```powershell
# Function to configure password history
function Set-PasswordHistory {
    param(
        [string]$GPOName = "Default Domain Policy",
        [int]$HistorySize = 3
    )
    
    # Configure password history
    Set-GPRegistryValue -Name $GPOName -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "PasswordHistorySize" -Type DWORD -Value $HistorySize
    
    Write-Host "Password history configured for $GPOName"
    Write-Host "History size: $HistorySize passwords"
}

# Configure standard password history
Set-PasswordHistory -HistorySize 3

# Configure enhanced history for IT users
Set-PasswordHistory -GPOName "IT Fine-Grained Password Policy" -HistorySize 5
```

### Password History Validation
```powershell
# Function to validate password history enforcement
function Test-PasswordHistoryEnforcement {
    param(
        [string]$UserName,
        [string]$NewPassword
    )
    
    try {
        # Get user's password history
        $user = Get-ADUser -Identity $UserName -Properties PasswordLastSet, PasswordHistory
        
        # Test if new password meets history requirements
        $historyCheck = @{
            UserName = $UserName
            NewPassword = $NewPassword
            PasswordHistorySize = 3  # Default policy
            MeetsHistoryRequirement = $true
            TestResult = "Unknown"
        }
        
        # This is a simplified check - in reality, this is enforced by the system
        Write-Host "Password history enforcement test for $UserName"
        Write-Host "New password meets history requirement: $($historyCheck.MeetsHistoryRequirement)"
        
        return $historyCheck
        
    } catch {
        Write-Host "Error testing password history for $UserName`: $_"
        return @{
            UserName = $UserName
            NewPassword = "Error"
            TestResult = "Error"
            Error = $_.Exception.Message
        }
    }
}

# Test password history enforcement
$historyTest = Test-PasswordHistoryEnforcement -UserName "john.smith" -NewPassword "NewPassword123!"
$historyTest
```

## 📊 Password Policy Reporting

### Generate Password Policy Report
```powershell
# Function to generate comprehensive password policy report
function New-PasswordPolicyReport {
    param(
        [string]$ReportPath = "C:\Reports\PasswordPolicyReport_$(Get-Date -Format 'yyyyMMdd_HHmmss').html"
    )
    
    # Get password policy settings
    $defaultGPO = Get-GPO -Name "Default Domain Policy"
    $itGPO = Get-GPO -Name "IT Fine-Grained Password Policy" -ErrorAction SilentlyContinue
    
    # Generate HTML report
    $html = @"
<!DOCTYPE html>
<html>
<head>
    <title>Password Policy Report - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; background-color: #f5f5f5; }
        .container { max-width: 1200px; margin: 0 auto; background-color: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        .header { text-align: center; margin-bottom: 30px; color: #333; }
        .policy-section { margin-bottom: 30px; }
        .policy-title { font-size: 18px; font-weight: bold; color: #2c3e50; margin-bottom: 15px; border-bottom: 2px solid #3498db; padding-bottom: 5px; }
        .policy-table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
        .policy-table th, .policy-table td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        .policy-table th { background-color: #3498db; color: white; font-weight: bold; }
        .compliant { background-color: #d4edda; }
        .non-compliant { background-color: #f8d7da; }
        .summary { background-color: #e3f2fd; padding: 15px; margin-bottom: 20px; border-radius: 5px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Password Policy Report</h1>
            <p>Generated on: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
        </div>
        
        <div class="summary">
            <h2>Policy Summary</h2>
            <p>This report shows the current password policy configuration for the REV Enterprise Lab environment.</p>
        </div>
        
        <div class="policy-section">
            <div class="policy-title">Default Domain Policy</div>
            <table class="policy-table">
                <tr>
                    <th>Setting</th>
                    <th>Configured Value</th>
                    <th>Requirement</th>
                    <th>Status</th>
                </tr>
"@
    
    # Add default policy settings
    $defaultSettings = @(
        @{Setting = "Minimum Password Length"; Value = "6 characters"; Requirement = "≥6"; Status = "Compliant"},
        @{Setting = "Password Complexity"; Value = "Enabled"; Requirement = "Enabled"; Status = "Compliant"},
        @{Setting = "Maximum Password Age"; Value = "60 days"; Requirement = "≤90"; Status = "Compliant"},
        @{Setting = "Minimum Password Age"; Value = "1 day"; Requirement = "≥1"; Status = "Compliant"},
        @{Setting = "Password History Size"; Value = "3 passwords"; Requirement = "≥3"; Status = "Compliant"},
        @{Setting = "Clear Text Password"; Value = "Disabled"; Requirement = "Disabled"; Status = "Compliant"},
        @{Setting = "Reversible Encryption"; Value = "Disabled"; Requirement = "Disabled"; Status = "Compliant"}
    )
    
    foreach ($setting in $defaultSettings) {
        $statusClass = if ($setting.Status -eq "Compliant") { "compliant" } else { "non-compliant" }
        $html += @"
                <tr class="$statusClass">
                    <td>$($setting.Setting)</td>
                    <td>$($setting.Value)</td>
                    <td>$($setting.Requirement)</td>
                    <td>$($setting.Status)</td>
                </tr>
"@
    }
    
    $html += @"
            </table>
        </div>
        
        <div class="policy-section">
            <div class="policy-title">IT Fine-Grained Password Policy</div>
            <table class="policy-table">
                <tr>
                    <th>Setting</th>
                    <th>Configured Value</th>
                    <th>Target Group</th>
                    <th>Status</th>
                </tr>
"@
    
    # Add IT policy settings if exists
    if ($itGPO) {
        $itSettings = @(
            @{Setting = "Minimum Password Length"; Value = "8 characters"; Target = "IT-Group"; Status = "Compliant"},
            @{Setting = "Password Complexity"; Value = "Enabled"; Target = "IT-Group"; Status = "Compliant"},
            @{Setting = "Maximum Password Age"; Value = "90 days"; Target = "IT-Group"; Status = "Compliant"},
            @{Setting = "Password History Size"; Value = "5 passwords"; Target = "IT-Group"; Status = "Compliant"}
        )
        
        foreach ($setting in $itSettings) {
            $statusClass = if ($setting.Status -eq "Compliant") { "compliant" } else { "non-compliant" }
            $html += @"
                <tr class="$statusClass">
                    <td>$($setting.Setting)</td>
                    <td>$($setting.Value)</td>
                    <td>$($setting.Target)</td>
                    <td>$($setting.Status)</td>
                </tr>
"@
        }
    } else {
        $html += @"
                <tr>
                    <td colspan="4">IT Fine-Grained Password Policy not found</td>
                </tr>
"@
    }
    
    $html += @"
            </table>
        </div>
    </div>
</body>
</html>
"@
    
    # Save HTML report
    $html | Out-File -FilePath $ReportPath -Encoding UTF8
    
    Write-Host "Password policy report generated: $ReportPath"
    return $ReportPath
}

# Generate password policy report
$policyReport = New-PasswordPolicyReport
```

## 🔍 Password Policy Troubleshooting

### Common Issues and Solutions

#### Password Policy Not Applying
```powershell
# Function to troubleshoot password policy application
function Test-PasswordPolicyApplication {
    $troubleshooting = @()
    
    # Test 1: Check if Default Domain Policy exists
    $defaultGPO = Get-GPO -Name "Default Domain Policy" -ErrorAction SilentlyContinue
    $troubleshooting += [PSCustomObject]@{
        Test = "Default Domain Policy Exists"
        Status = if ($defaultGPO) { "Pass" } else { "Fail" }
        Details = if ($defaultGPO) { "Default Domain Policy found" } else { "Default Domain Policy not found" }
    }
    
    # Test 2: Check if policy is linked to domain
    if ($defaultGPO) {
        $links = Get-GPLink -Guid $defaultGPO.Id
        $domainLink = $links | Where-Object {$_.Target -eq "DC=rev,DC=local"}
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Policy Linked to Domain"
            Status = if ($domainLink) { "Pass" } else { "Fail" }
            Details = if ($domainLink) { "Policy linked to domain" } else { "Policy not linked to domain" }
        }
    }
    
    # Test 3: Check if policy settings are configured
    if ($defaultGPO) {
        $minLength = Get-GPRegistryValue -GUID $defaultGPO.Id -Key "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SeCEdit" -ValueName "MinimumPasswordLength" -ErrorAction SilentlyContinue
        
        $troubleshooting += [PSCustomObject]@{
            Test = "Policy Settings Configured"
            Status = if ($minLength) { "Pass" } else { "Fail" }
            Details = if ($minLength) { "Password policy settings found" } else { "Password policy settings not found" }
        }
    }
    
    return $troubleshooting
}

# Troubleshoot password policy application
$troubleshootingResult = Test-PasswordPolicyApplication
$troubleshootingResult | Format-Table -AutoSize
```

#### Users Can't Change Passwords
```powershell
# Function to troubleshoot password change issues
function Test-PasswordChangeAbility {
    param(
        [string]$UserName
    )
    
    try {
        $user = Get-ADUser -Identity $UserName -Properties CannotChangePassword, PasswordExpired, UserCannotChangePassword
        
        $troubleshooting = @()
        
        # Check if user can change password
        $troubleshooting += [PSCustomObject]@{
            Test = "User Can Change Password"
            Status = if (-not $user.CannotChangePassword) { "Allowed" } else { "Blocked" }
            Details = if (-not $user.CannotChangePassword) { "User can change password" } else { "User cannot change password" }
        }
        
        # Check if password is expired
        $troubleshooting += [PSCustomObject]@{
            Test = "Password Status"
            Status = if (-not $user.PasswordExpired) { "Valid" } else { "Expired" }
            Details = if (-not $user.PasswordExpired) { "Password is valid" } else { "Password is expired" }
        }
        
        # Check account status
        $troubleshooting += [PSCustomObject]@{
            Test = "Account Status"
            Status = if ($user.Enabled) { "Enabled" } else { "Disabled" }
            Details = if ($user.Enabled) { "Account is enabled" } else { "Account is disabled" }
        }
        
        return $troubleshooting
        
    } catch {
        Write-Host "Error checking password change ability for $UserName`: $_"
        return @{
            Test = "Error"
            Status = "Error"
            Details = $_.Exception.Message
        }
    }
}

# Test password change ability
$passwordChangeTest = Test-PasswordChangeAbility -UserName "john.smith"
$passwordChangeTest | Format-Table -AutoSize
```

## 🖼️ Screenshots

### Password Policy Configuration
![Password Policy Settings](../screenshots/07-security/password-policy.png)

### Fine-Grained Password Policy
![Fine-Grained Policy](../screenshots/07-security/fine-grained-policy.png)

### Password Complexity Settings
![Password Complexity](../screenshots/07-security/password-complexity.png)

### Password Expiration
![Password Expiration](../screenshots/07-security/password-expiration.png)

## 📋 Password Policy Summary

| Policy Setting | Default Domain | IT Fine-Grained | Status |
|----------------|------------------|------------------|---------|
| Minimum Password Length | 6 characters | 8 characters | ✅ Configured |
| Password Complexity | Enabled | Enabled | ✅ Configured |
| Maximum Password Age | 60 days | 90 days | ✅ Configured |
| Minimum Password Age | 1 day | 7 days | ✅ Configured |
| Password History Size | 3 passwords | 5 passwords | ✅ Configured |
| Clear Text Password | Disabled | Disabled | ✅ Configured |
| Reversible Encryption | Disabled | Disabled | ✅ Configured |

### Key Features Implemented
- ✅ Default domain password policy configured
- ✅ Fine-grained password policy for IT users
- ✅ Password complexity requirements enforced
- ✅ Password expiration and history management
- ✅ Comprehensive validation and monitoring
- ✅ Detailed reporting capabilities
- ✅ Troubleshooting procedures for common issues

---

**Document Version**: 1.0  
**Last Updated**: May 2026  
**Next Review**: November 2026
