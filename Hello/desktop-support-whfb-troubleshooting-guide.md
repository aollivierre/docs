# Windows Hello for Business Troubleshooting Guide for Desktop Support
## For Windows 11 Enterprise Systems (Hybrid Joined & Co-managed)

### Administrative Privilege Requirements
> ⚠️ **IMPORTANT**: Some WHfB troubleshooting commands require specific privileges. Here's a breakdown:
>
> **Always Requires Admin:**
> - TPM operations (Get-Tpm, Clear-Tpm)
> - NGC folder modifications (certutil -DeleteHelloContainer)
> - Registry modifications in HKLM
> - System-wide event logs
>
> **User-Level (No Admin Required):**
> - Device registration status (dsregcmd /status)
> - Viewing current user's certificates
> - Basic event log viewing (HelloForBusiness/Operational)
> - PIN management via Settings
> - Viewing current user's NGC status
>
> **May Require Admin (Context-Dependent):**
> - Azure AD operations (depends on user's Azure AD role)
> - Some event logs (depends on log permissions)
> - Certificate operations (depends on certificate store)

### Prerequisites Check
Before troubleshooting, verify these requirements using PowerShell:

```powershell
# === USER LEVEL (No Admin Required) ===
# Check device registration status
dsregcmd /status | findstr /i "AzureAdJoined DomainJoined"

# Check user's certificates
Get-ChildItem -Path Cert:\CurrentUser\My -EKU "1.3.6.1.4.1.311.20.2.2"

# View user's WHfB events
Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational" -MaxEvents 5

# === REQUIRES ADMIN ===
# Verify TPM status
Get-Tpm
Get-TpmEndorsementKeyInfo

# Check Windows Hello for Business system status
certutil -generateHelloContainer -status
```

### Common Issues and PowerShell Diagnostic Commands

#### 1. Device Registration Issues
```powershell
# === USER LEVEL (No Admin Required) ===
# Check device registration status
dsregcmd /status

# === REQUIRES ADMIN ===
# Check device in Azure AD (requires appropriate Azure AD role)
Get-AzureADDevice -SearchString $env:COMPUTERNAME

# Verify NGC folder exists and permissions
$ngcPath = "$env:SystemDrive\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc"
Test-Path $ngcPath
Get-Acl $ngcPath

# Check registry settings
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\PolicyManager\default\Authentication\EnablePinSignIn"
```

#### 2. Certificate Problems
```powershell
# === USER LEVEL (No Admin Required) ===
# List Windows Hello certificates
certutil -store -user My

# Check certificate template
$cert = Get-ChildItem -Path Cert:\CurrentUser\My -EKU "1.3.6.1.4.1.311.20.2.2"
$cert | Format-List Subject, Issuer, NotBefore, NotAfter, TemplateInformation

# === REQUIRES ADMIN ===
# Verify certificate chain
certutil -verify $cert.Thumbprint

# Check Smart Card KSP status
certutil -csp "Microsoft Smart Card Key Storage Provider" -key -v

# Verify enterprise certificate settings
certutil -enterpriseCA
```

#### 3. NGC (Next Generation Credentials) Issues
```powershell
# === REQUIRES ADMIN ===
# Take ownership of NGC folder
$ngcPath = "$env:SystemDrive\Windows\ServiceProfiles\LocalService\AppData\Local\Microsoft\Ngc"
takeown /f $ngcPath /r /d y
icacls $ngcPath /grant administrators:F /t

# Clean NGC folder (requires restart)
certutil -DeleteHelloContainer

# Reset NGC state
Remove-Item -Path $ngcPath -Force -Recurse

# Check NGC key type (1=TPM, 2=Software)
certutil -csp "Microsoft Passport Key Storage Provider" -key -v | findstr NgcKeyImplType

# Verify NGC container status
certutil -generateHelloContainer -status
```

#### 4. TPM Troubleshooting
```powershell
# === REQUIRES ADMIN ===
# Detailed TPM status
Get-Tpm | Select-Object -Property *

# Check TPM ownership
(Get-Tpm).OwnerClearDisabled

# Verify TPM is ready for WHfB
Get-WinEvent -LogName "Microsoft-Windows-TPM-WMI/Operational"

# Check TPM provisioning status
Get-CimInstance -Namespace root/cimv2/Security/MicrosoftTpm -ClassName Win32_Tpm

# Verify TPM PCR status
Get-TpmEndorsementKeyInfo
```

#### 5. Event Log Analysis
```powershell
# === USER LEVEL (No Admin Required) ===
# Windows Hello for Business operational logs
Get-WinEvent -LogName "Microsoft-Windows-HelloForBusiness/Operational"

# === REQUIRES ADMIN ===
# System-wide authentication logs
Get-WinEvent -LogName "Microsoft-Windows-Authentication/AuthenticationPolicyFailures-DomainController"

# TPM errors
Get-WinEvent -LogName "Microsoft-Windows-TPM-WMI/Admin"

# Export all relevant logs
$logPaths = @(
    "Microsoft-Windows-HelloForBusiness/Operational",
    "Microsoft-Windows-User Device Registration/Admin",
    "Microsoft-Windows-TPM-WMI/Operational"
)
foreach ($log in $logPaths) {
    $fileName = ($log -split '/')[-1]
    Get-WinEvent -LogName $log | Export-Csv -Path "$env:USERPROFILE\Desktop\${fileName}_Logs.csv"
}
```

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
