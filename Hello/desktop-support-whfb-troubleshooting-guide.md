# Windows Hello for Business Troubleshooting Guide for Desktop Support
## For Windows 11 Enterprise Systems (Hybrid Joined & Co-managed)

### Key Concepts
> **Important Terms**:
> - **NGC (Next Generation Credentials)**: The underlying technology for Windows Hello for Business that provides secure storage and management of modern credentials, replacing traditional passwords.
> - **TPM (Trusted Platform Module)**: A hardware-based security component that securely stores authentication keys.
> - **Windows Hello for Business**: Enterprise-grade authentication that uses NGC technology with biometrics or PINs.

### Domain Environment Requirements
> **IMPORTANT**: In a domain environment (Active Directory), Windows Hello for Business requires specific configuration:
>
> **Domain Level Requirements:**
> - Group Policy configuration for Windows Hello for Business
> - Certificate templates (for certificate trust deployment)
> - Or key trust deployment configuration
> - Domain Controller configuration
>
> **Local Machine Requirements:**
> - TPM 2.0
> - Windows 11 Enterprise
> - Proper NGC service configuration
> - Domain joined status

### Administrative Privilege Requirements
> **IMPORTANT**: Some WHfB troubleshooting commands require specific privileges. Here's a breakdown:
>
> **Always Requires Admin:**
> - TPM operations (Get-Tpm, Get-TpmEndorsementKeyInfo)
> - NGC folder modifications
> - Registry modifications in HKLM
> - System-wide event logs
>
> **User-Level (No Admin Required):**
> - Device registration status (dsregcmd /status)
> - Viewing current user's certificates
> - Basic event log viewing (HelloForBusiness/Operational)
> - PIN management via Settings
>
> **May Require Domain Admin:**
> - Group Policy modifications
> - Certificate template configuration
> - Domain Controller configuration

### Pre-Setup Diagnostics
Before Windows Hello for Business is configured, verify these requirements using PowerShell:

```powershell
# === USER LEVEL (No Admin Required) ===
# Check device registration and domain status
dsregcmd /status | findstr /i "AzureAdJoined DomainJoined"
systeminfo | findstr /B /C:"Domain"

# View WHfB events (works before and after setup)
Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational" -MaxEvents 5

# === REQUIRES ADMIN ===
# Verify TPM status
Get-Tpm | Select-Object TpmPresent, TpmReady, TpmEnabled, TpmActivated

# Check TPM endorsement key
Get-TpmEndorsementKeyInfo

# Verify NGC infrastructure
$ngcPath = "$env:SystemDrive\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc"
Test-Path $ngcPath
Get-Acl $ngcPath | Format-List

# Check Windows Hello credential provider registration
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{D6886603-9D2F-4EB2-B667-1971041FA96B}"

# Check NGC services status
Get-Service NgcSvc, NgcCtnrSvc | Select-Object Name, Status, StartType

# Verify Windows Hello capabilities
Get-WindowsCapability -Online | Where-Object { $_.Name -like "*Hello*" }
```

### Domain Configuration Checks
For domain environments, verify these additional items:

```powershell
# Check domain controller availability
nltest /dsgetdc:$env:USERDNSDOMAIN

# Check Group Policy settings (requires Domain Admin)
gpresult /h "$env:USERPROFILE\Desktop\GPReport.html"

# Check local security policy
secedit /export /cfg "$env:USERPROFILE\Desktop\secpol.cfg"
Get-Content "$env:USERPROFILE\Desktop\secpol.cfg" | Select-String -Pattern "Pin|Hello|Credential|NGC"

# Check Windows Hello policies
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork" -ErrorAction SilentlyContinue
```

### Post-Setup Verification
After Windows Hello for Business is configured with a PIN or biometrics, additional diagnostics become available:

```powershell
# === USER LEVEL (No Admin Required) ===
# Check for WHfB certificates
Get-ChildItem -Path Cert:\CurrentUser\My -EKU "1.3.6.1.4.1.311.20.2.2"

# === REQUIRES ADMIN ===
# Check NGC key storage
certutil -csp "Microsoft Passport Key Storage Provider" -key -v

# Verify NGC container contents
Get-ChildItem -Path $ngcPath -Recurse -Force
```

### Common Issues and Solutions

#### 1. TPM Issues
If TPM checks fail:
- Verify TPM is enabled in BIOS
- Check TPM status: `Get-Tpm | Select-Object -Property *`
- Review TPM events in Event Viewer

#### 2. NGC (Next Generation Credentials) Issues
If NGC checks fail:
- Verify NGC services are running: `Get-Service NgcSvc, NgcCtnrSvc`
- Check NGC folder permissions (should be owned by LOCAL SERVICE)
- Review NGC events in Event Viewer

#### 3. Domain-Related Issues
If Windows Hello options are not appearing:
- Verify Group Policy settings
- Check domain connectivity
- Ensure proper certificate templates are configured (for certificate trust)
- Verify domain controller configuration

### Event Log Analysis
```powershell
# === USER LEVEL (No Admin Required) ===
# Windows Hello for Business operational logs
Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational" -MaxEvents 10

# === REQUIRES ADMIN ===
# Export WHfB logs for analysis
$logPath = "$env:USERPROFILE\Desktop\WHfB_Logs.csv"
Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational" | Export-Csv -Path $logPath
```

### Troubleshooting Tips
1. **Pre-Setup Issues**:
   - Verify TPM is ready and enabled
   - Ensure NGC services are running
   - Check device registration status
   - Verify domain connectivity

2. **Domain Environment**:
   - Group Policy settings take precedence
   - Local configuration may be overridden by domain policy
   - Certificate or key trust must be properly configured

3. **General Tips**:
   - Always check both user-level and admin-level components
   - Review event logs for sequence of operations
   - Verify service dependencies are running
   - In VMs, some biometric features may not be available

> **Note**: Some commands will show limited or no output before Windows Hello for Business is configured. This is normal and expected behavior.

### Advanced Health Check Script
```powershell
# === REQUIRES ADMIN ===
# Save this as Check-WHfBHealth.ps1
function Check-WHfBHealth {
    # Verify running as admin
    if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
        Write-Host "Please run as Administrator!" -ForegroundColor Red
        return
    }

    Write-Host "=== Windows Hello for Business Health Check ===" -ForegroundColor Green
    
    # Check Device Registration
    Write-Host "`nChecking Device Registration..." -ForegroundColor Yellow
    dsregcmd /status | Select-String "AzureAdJoined|DomainJoined"
    
    # Check TPM
    Write-Host "`nChecking TPM Status..." -ForegroundColor Yellow
    $tpm = Get-Tpm
    $tpm | Format-List IsEnabled, IsActivated, IsOwned, RestartPending
    
    # Check NGC
    Write-Host "`nChecking NGC Status..." -ForegroundColor Yellow
    $ngcPath = "$env:SystemDrive\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc"
    if (Test-Path $ngcPath) {
        Write-Host "NGC folder exists" -ForegroundColor Green
        # Check NGC container status
        $ngcStatus = certutil -generateHelloContainer -status
        Write-Host "NGC Container Status:" -ForegroundColor Yellow
        Write-Host $ngcStatus
    } else {
        Write-Host "NGC folder missing!" -ForegroundColor Red
    }
    
    # Check Certificates
    Write-Host "`nChecking Certificates..." -ForegroundColor Yellow
    $certs = Get-ChildItem -Path Cert:\CurrentUser\My -EKU "1.3.6.1.4.1.311.20.2.2"
    if ($certs) {
        Write-Host "Found $($certs.Count) WHfB certificates" -ForegroundColor Green
        $certs | Format-Table Subject, NotAfter, Thumbprint
        
        # Verify certificate chains
        foreach ($cert in $certs) {
            Write-Host "`nVerifying certificate chain for: $($cert.Subject)" -ForegroundColor Yellow
            certutil -verify $cert.Thumbprint
        }
    } else {
        Write-Host "No WHfB certificates found!" -ForegroundColor Red
    }
    
    # Check Policy Settings
    Write-Host "`nChecking WHfB Policy Settings..." -ForegroundColor Yellow
    $policyPath = "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork"
    if (Test-Path $policyPath) {
        Get-ItemProperty -Path $policyPath | Format-Table -AutoSize
    } else {
        Write-Host "No WHfB policies found!" -ForegroundColor Red
    }
    
    # Check Recent Events
    Write-Host "`nChecking Recent WHfB Events..." -ForegroundColor Yellow
    Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational" -MaxEvents 5 |
        Format-Table TimeCreated, Id, LevelDisplayName, Message -Wrap
}

# Run the health check
Check-WHfBHealth
```

### Reset and Removal Procedures

#### Non-Destructive Options
These options preserve user data and system configuration:

```powershell
# === USER LEVEL (No Admin Required) ===
# 1. Reset PIN (through Settings UI)
ms-settings:signinoptions   # Open Sign-in Options

# 2. View current PIN reset count
Get-ItemProperty -Path "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{D6886603-9D2F-4EB2-B667-1971041FA96B}" -ErrorAction SilentlyContinue

# === REQUIRES ADMIN ===
# 3. Reset PIN for specific user (less destructive)
$username = "username"
$sid = (Get-WmiObject -Class Win32_UserAccount -Filter "Name='$username'").SID
Remove-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{D6886603-9D2F-4EB2-B667-1971041FA96B}\$sid" -Force -ErrorAction SilentlyContinue
```

#### Moderately Destructive Options
These options remove Windows Hello settings but preserve other configurations:

```powershell
# === REQUIRES ADMIN ===
# 1. Remove Windows Hello container using certutil
certutil -deletehellocontainer

# Note: Expected errors if Windows Hello is not configured:
# "CertUtil: -DeleteHelloContainer command FAILED: 0x80090010 (-2146893808 NTE_PERM)"
# "CertUtil: Access denied."

# Alternative method: Remove Windows Hello container manually
$containerPath = "$env:SystemDrive\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc"
Get-ChildItem -Path $containerPath -Filter "*$env:USERNAME*" | Remove-Item -Force -Recurse

# 2. Stop and reset NGC services (temporary)
Stop-Service NgcSvc, NgcCtnrSvc -Force
Start-Service NgcSvc, NgcCtnrSvc

# 3. Clear TPM keys for Windows Hello (preserves other TPM data)
Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational" -MaxEvents 1000 | 
    Where-Object { $_.Id -eq 300 } | 
    ForEach-Object { 
        $ngcPath = "$containerPath\$($_.Properties[0].Value)"
        if (Test-Path $ngcPath) {
            Remove-Item -Path $ngcPath -Force -Recurse
        }
    }
```

#### Destructive Options (Use with Caution)
These options completely remove Windows Hello and related configurations:

```powershell
# === REQUIRES ADMIN ===
# 1. Remove all NGC containers (affects all users)
Stop-Service NgcSvc, NgcCtnrSvc -Force
Remove-Item -Path "$env:SystemDrive\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc" -Force -Recurse
Start-Service NgcSvc, NgcCtnrSvc

# 2. Clear TPM for Windows Hello (affects other TPM-dependent features)
Clear-Tpm

# 3. Remove Windows Hello policies
Remove-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\PassportForWork" -Force -Recurse
Remove-Item -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\System\AllowDomainPINLogon" -Force

# 4. Disable Windows Hello services (most destructive)
Set-Service NgcSvc -StartupType Disabled
Set-Service NgcCtnrSvc -StartupType Disabled
Stop-Service NgcSvc, NgcCtnrSvc -Force
```

> ⚠️ **WARNING**:
> - Always backup important data before performing destructive operations
> - In domain environments, these changes might be reverted by Group Policy
> - Clearing TPM affects other security features like BitLocker
> - Document current settings before making destructive changes
> - Consider user impact and schedule maintenance window if needed

#### Post-Reset Steps
After performing any reset operation:

1. Non-Destructive Reset:
   - User can set up new PIN immediately
   - No system restart required
   - No impact on other users

2. Moderate Reset:
   - Sign out and sign back in
   - Set up Windows Hello again
   - Other users not affected

3. Destructive Reset:
   - Restart computer
   - Reconfigure Windows Hello policies
   - All users must set up Windows Hello again
   - Verify TPM status if cleared
   - Check BitLocker and other security features

### When to Escalate
Escalate to Tier 2 support if:
- TPM shows errors or is not detected
- Certificate chain validation fails
- Multiple users report similar issues
- Health check script shows multiple failures
- Admin-level commands reveal system-wide issues

### Required Information for Escalation
When escalating, provide:
1. Output of health check script
2. Event logs (export using provided PowerShell commands)
3. Certificate details
4. TPM status
5. Device registration status
6. Registry settings for WHfB policies

### Best Practices for Support
1. Always verify admin privileges before running commands
2. Document all commands run and their output
3. Export logs before making any changes
4. Use the health check script as first diagnostic step
5. Never clear TPM without backing up important data
6. Test in both user and admin contexts when appropriate
7. Verify policy settings in registry before making changes

### Quick Reference: Common Error Codes
- 0x8004425: TPM not ready (Requires Admin)
- 0x80090016: Bad PIN (User-Level)
- 0x80090029: NGC container error (Requires Admin)
- 0x80090035: Device registration issue (Requires Admin)
- 0x801C03ED: TPM/NGC corruption (Requires Admin)

### Contact Information
[Insert your organization's specific contact details]

---
**Note:** This guide is specifically for certificate trust-based Windows Hello for Business. Commands marked as "Requires Admin" must be run from an elevated PowerShell prompt.
