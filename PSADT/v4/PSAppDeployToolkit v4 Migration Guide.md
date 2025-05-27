# PSAppDeployToolkit v4 Migration Guide

## Table of Contents

1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Architecture Changes](#architecture-changes)
4. [Creating a New Deployment](#creating-a-new-deployment)
5. [Command Line Migration](#command-line-migration)
6. [Deployment Modes](#deployment-modes)
7. [Deferral Configuration](#deferral-configuration)
8. [Deferral Timing Mechanism](#deferral-timing-mechanism)
9. [Organization Branding](#organization-branding)
10. [Extension Modules](#extension-modules)
11. [Best Practices](#best-practices)

## Introduction

PSAppDeployToolkit v4 represents a significant architectural change from v3, introducing session-based deployments, improved PowerShell practices, and enhanced user interface options. This guide covers the key differences and migration steps for organizations transitioning from v3 to v4.

## Installation

### PowerShell Gallery (Recommended)
```powershell
Install-Module -Name PSAppDeployToolkit -Scope CurrentUser
```

### Manual Download
Download from [PSAppDeployToolkit Latest Release](https://github.com/PSAppDeployToolkit/PSAppDeployToolkit/releases) and extract the ZIP files.

### Available Downloads
| Filename | Description |
|----------|-------------|
| PSAppDeployToolkit.zip | Main module with all core functions |
| PSAppDeployToolkit_Template_v3.zip | v3 compatibility template |
| PSAppDeployToolkit_Template_v4.zip | v4 native template |

## Architecture Changes

### Key Differences from v3

| Feature | v3 | v4 |
|---------|----|----|
| **Architecture** | Monolithic script | Session-based functions |
| **Structure** | Deploy-Application.ps1 | Invoke-AppDeployToolkit.ps1 |
| **Functions** | Inline code blocks | Dedicated Install/Uninstall/Repair functions |
| **Session Management** | None | Open-ADTSession / Close-ADTSession |
| **Function Invocation** | If/else blocks | Dynamic function calling |

### Dynamic Function Invocation

v4 uses dynamic function calls instead of conditional blocks:

```powershell
# v4 approach - calls function based on deployment type
& "$($adtSession.DeploymentType)-ADTDeployment"

# Dynamically calls:
# Install-ADTDeployment
# Uninstall-ADTDeployment  
# Repair-ADTDeployment
```

## Creating a New Deployment

### Template Creation
```powershell
# Create new v4 deployment template
New-ADTTemplate -Name "7Zip-Deployment" -Path "C:\Deployments"
```

### Directory Structure
```
7Zip-Deployment/
├── Files/                          # Application files (MSI, EXE, etc.)
├── Assets/                         # Images, logos, banners
├── Config/                         # Configuration files
├── PSAppDeployToolkit/             # Main toolkit module
├── PSAppDeployToolkit.Extensions/  # Custom extensions
└── Invoke-AppDeployToolkit.ps1     # Main deployment script
```

### Basic Configuration Example

```powershell
$adtSession = @{
    AppVendor = '7-Zip'
    AppName = '7-Zip'
    AppVersion = '24.09'
    AppArch = 'x64'
    AppLang = 'EN'
    InstallName = '7-Zip 24.09 x64'
    InstallTitle = '7-Zip 24.09 x64 Installation'
}
```

### Installation Function Example

```powershell
function Install-ADTDeployment {
    Show-ADTInstallationWelcome -CloseProcesses '7zFM,7zG' -AllowDefer -DeferTimes 3
    Start-ADTMsiProcess -Action 'Install' -FilePath '7z2409-x64.msi' -ArgumentList '/QN'
    Show-ADTInstallationPrompt -Message 'Installation completed successfully.' -ButtonRightText 'OK'
}
```

## Command Line Migration

### v3 to v4 Command Changes

| Component | v3 | v4 |
|-----------|----|----|
| **Executable Name** | Deploy-Application.exe | Invoke-AppDeployToolkit.exe |
| **Core Parameters** | Same | Same |
| **New Parameters** | N/A | -TerminalServerMode, -DisableLogging |

### SCCM/Intune Migration

#### v3 Commands
```json
{
    "installCommandLine": "\"Deploy-Application.exe\" -DeploymentType Install",
    "uninstallCommandLine": "\"Deploy-Application.exe\" -DeploymentType uninstall"
}
```

#### v4 Commands
```json
{
    "installCommandLine": "\"Invoke-AppDeployToolkit.exe\" -DeploymentType Install -DeployMode Silent",
    "uninstallCommandLine": "\"Invoke-AppDeployToolkit.exe\" -DeploymentType Uninstall -DeployMode Silent",
    "repairCommandLine": "\"Invoke-AppDeployToolkit.exe\" -DeploymentType Repair -DeployMode Silent"
}
```

## Deployment Modes

### Mode Comparison

| Feature | Interactive | NonInteractive | Silent |
|---------|-------------|----------------|--------|
| **User Dialogs** | ✅ Full | ✅ Info Only | ❌ None |
| **User Input** | ✅ Required | ❌ Auto-proceed | ❌ None |
| **Deferrals** | ✅ Available | ❌ Not Available | ❌ Not Available |
| **Progress Display** | ✅ Yes | ✅ Yes | ❌ No |
| **Process Closure** | User Choice | Automatic | Automatic |
| **Best For** | End Users | Kiosks | Automation |

### Usage Examples

#### Interactive Mode
```powershell
.\Invoke-AppDeployToolkit.exe -DeploymentType Install -DeployMode Interactive
```
- Shows all dialogs with user interaction
- Allows deferrals and user choices
- Waits for user input

#### Silent Mode  
```powershell
.\Invoke-AppDeployToolkit.exe -DeploymentType Install -DeployMode Silent
```
- No dialogs shown
- Completely unattended
- Ideal for SCCM/Intune

#### NonInteractive Mode
```powershell
.\Invoke-AppDeployToolkit.exe -DeploymentType Install -DeployMode NonInteractive
```
- Shows informational dialogs
- No user input required
- Auto-set for SYSTEM context

## Deferral Configuration

### Available Deferral Options

```powershell
Show-ADTInstallationWelcome -CloseProcesses 'notepad,calc' -AllowDefer -DeferTimes 5 -DeferDays 7 -DeferDeadline '2025-02-15' -CloseProcessesCountdown 600
```

### Deferral Parameters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| **DeferTimes** | UInt32 | Number of deferrals allowed | 3, 5, 10 |
| **DeferDays** | UInt32 | Days since first run | 7, 14, 30 |
| **DeferDeadline** | String | Specific deadline date | '2025-02-15 18:00:00' |
| **CloseProcessesCountdown** | UInt32 | Countdown before closing apps | 300, 600 |
| **ForceCloseProcessesCountdown** | UInt32 | Force countdown timer | 60, 120 |

### Configuration Examples

#### Standard Deferral

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferTimes 3
```

#### Time-Based Deferral

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferDays 7
```

#### Deadline-Based Deferral

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferDeadline '2025-02-15'
```

#### Combined Approach

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferTimes 5 -DeferDeadline '2025-02-15' -CloseProcessesCountdown 600
```

### DeferDays vs DeferDeadline - The Key Difference

These two parameters control **when deferrals expire**, but they work in completely different ways:

#### 🗓️ DeferDays - Relative Time (Dynamic)

```powershell
-DeferDays 7
```

**How it works:**

- ✅ **Starts counting from FIRST RUN** of the deployment
- ✅ **Dynamic deadline** - calculated when script first executes
- ✅ **Relative to deployment attempt**

**Example Scenario:**

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferDays 7
```

**Timeline:**

- **January 20** - User runs deployment for first time → Deadline set to **January 27**
- **January 22** - User defers → Still has until **January 27**
- **January 25** - User defers → Still has until **January 27**
- **January 28** - User tries to defer → **❌ TOO LATE** - Installation proceeds

#### 📅 DeferDeadline - Absolute Time (Fixed)

```powershell
-DeferDeadline '2025-02-15 18:00:00'
```

**How it works:**

- ✅ **Fixed date/time** - same for everyone
- ✅ **Absolute deadline** - doesn't matter when first run
- ✅ **Organization-controlled**

**Example Scenario:**

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferDeadline '2025-02-15 18:00:00'
```

**Timeline:**

- **January 20** - User A runs deployment → Deadline is **February 15, 6 PM**
- **January 30** - User B runs deployment → Same deadline: **February 15, 6 PM**
- **February 10** - Both users can still defer → Until **February 15, 6 PM**
- **February 16** - Anyone tries to defer → **❌ TOO LATE** - Installation proceeds

#### 📊 Practical Comparison

**Scenario: IT wants everyone updated by February 15th**

**❌ Wrong Approach - DeferDays:**

```powershell
# BAD: Different deadlines for different users!
Show-ADTInstallationWelcome -AllowDefer -DeferDays 14
```

- User runs on **Jan 20** → Deadline **Feb 3** ✅
- User runs on **Feb 1** → Deadline **Feb 15** ✅
- User runs on **Feb 10** → Deadline **Feb 24** ❌ **TOO LATE!**

**✅ Correct Approach - DeferDeadline:**

```powershell
# GOOD: Same deadline for everyone!
Show-ADTInstallationWelcome -AllowDefer -DeferDeadline '2025-02-15 18:00:00'
```

- **ALL users** must install by **Feb 15, 6 PM** regardless of when they first see it

#### 🔄 Can You Use Both Together?

**YES!** You can combine them for **belt-and-suspenders** approach:

```powershell
Show-ADTInstallationWelcome -AllowDefer -DeferTimes 5 -DeferDays 14 -DeferDeadline '2025-02-15 18:00:00'
```

**Logic:** User can defer if **ALL conditions** are met:

- ✅ Still has deferral attempts left (5 max)
- ✅ **AND** hasn't exceeded 14 days since first run
- ✅ **AND** current date is before February 15th

**Whichever expires FIRST ends the deferral period.**

#### 🎯 Real-World Use Cases

**1. Patch Tuesday Scenario (DeferDeadline)**

```powershell
# Everyone must install by Friday after Patch Tuesday
Show-ADTInstallationWelcome -AllowDefer -DeferDeadline '2025-02-14 17:00:00'
```

**2. Rolling Deployment (DeferDays)**

```powershell
# Give each user 2 weeks from when they first see it
Show-ADTInstallationWelcome -AllowDefer -DeferDays 14
```

**3. Compliance Requirement (Combined)**

```powershell
# Must install within 7 days OR by month-end (whichever comes first)
Show-ADTInstallationWelcome -AllowDefer -DeferDays 7 -DeferDeadline '2025-01-31 23:59:59'
```

#### ⚠️ Important Notes

**DeferDays Tracking:**

- Uses **registry** to remember first run date
- Stored per-application (your specific deployment)
- Survives reboots and re-runs

**DeferDeadline Format:**

```powershell
# Various accepted formats:
-DeferDeadline '2025-02-15'                    # Date only (midnight)
-DeferDeadline '2025-02-15 18:00:00'          # Date + time
-DeferDeadline '02/15/2025'                   # US format
-DeferDeadline '2025-02-15T18:00:00Z'         # UTC format
```

#### 🔧 Implementation Recommendations

**Option 1: Give users 1 week from first encounter**

```powershell
Show-ADTInstallationWelcome -CloseProcesses '7zFM,7zG' -AllowDefer -DeferTimes 3 -DeferDays 7 -CheckDiskSpace -PersistPrompt
```

**Option 2: Hard deadline approach**

```powershell
Show-ADTInstallationWelcome -CloseProcesses '7zFM,7zG' -AllowDefer -DeferTimes 3 -DeferDeadline '2025-02-28 17:00:00' -CheckDiskSpace -PersistPrompt
```

**Option 3: Combined approach for maximum flexibility**

```powershell
Show-ADTInstallationWelcome -CloseProcesses '7zFM,7zG' -AllowDefer -DeferTimes 3 -DeferDays 7 -DeferDeadline '2025-02-28 17:00:00' -CheckDiskSpace -PersistPrompt
```

## Deferral Timing Mechanism

### Important Limitation

**PSADT v4 does NOT have automatic timing between deferrals.** The system is event-driven, not time-driven.

### When Deferrals Are Checked

- Script execution starts
- Manual trigger by administrator
- Scheduled task execution
- SCCM/Intune retry attempts

### Implementing Automatic Re-Prompting

#### Scheduled Task Approach
```powershell
$Trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Hours 4)
$Action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File 'C:\Path\To\Deployment\Invoke-AppDeployToolkit.ps1' -DeploymentType Install -DeployMode Interactive"
Register-ScheduledTask -TaskName "App-Deployment-Retry" -Trigger $Trigger -Action $Action
```

## Organization Branding

### Logo Configuration

Place your organization logo in the `Assets` directory and update the configuration:

```powershell
# Config/config.psd1
Assets = @{
    Logo = '..\Assets\org-logo.png'
    Banner = '..\Assets\Banner.Classic.png'
}

# Update balloon title
BalloonTitle = 'Your Organization IT'
```

### Logo Specifications

| Property | Recommendation |
|----------|----------------|
| **Format** | PNG (supports transparency) |
| **Size** | 256x256 pixels or higher |
| **Aspect Ratio** | Square (1:1) preferred |
| **File Size** | Under 2MB |

### UI Style Options

```powershell
# Modern Fluent UI (default)
DialogStyle = 'Fluent'

# Classic v3-style dialogs
DialogStyle = 'Classic'
```

## Extension Modules

### Purpose of Extensions

The `PSAppDeployToolkit.Extensions` module provides:

- Site-specific custom functions
- Reusable code across deployments  
- Organization-specific workflows
- Local customizations without modifying core toolkit

### Extension Structure

```
PSAppDeployToolkit.Extensions/
├── PSAppDeployToolkit.Extensions.psd1    # Module manifest
├── PSAppDeployToolkit.Extensions.psm1    # Module functions
└── Private/                              # Private functions
```

### Automatic Loading

Extensions are automatically imported alongside the main module:

```powershell
Get-Item -Path $PSScriptRoot\PSAppDeployToolkit.* | & {
    process {
        Import-Module -Name $_.FullName -Force
    }
}
```

### Custom Function Example

```powershell
function Install-ADTOrgSpecificSoftware {
    param([string]$SoftwareName)
    
    Write-ADTLogEntry "Installing $SoftwareName per organization standards"
    # Custom installation logic here
}
```

## Best Practices

### Migration Strategy

1. **Install PSADT v4** alongside existing v3 deployments
2. **Create test deployments** using v4 templates
3. **Validate functionality** in lab environment
4. **Update command lines** in deployment tools
5. **Migrate deployments incrementally**

### Configuration Management

- Store organization logos in source control
- Maintain consistent configuration across deployments
- Use Extensions module for reusable functions
- Document custom functions and workflows

### Testing Approach

```powershell
# Test in different modes
.\Invoke-AppDeployToolkit.exe -DeploymentType Install -DeployMode Interactive
.\Invoke-AppDeployToolkit.exe -DeploymentType Install -DeployMode Silent
.\Invoke-AppDeployToolkit.exe -DeploymentType Uninstall -DeployMode Silent
```

### Deployment Tool Integration

- Update SCCM/Intune command lines
- Test detection methods
- Validate return codes
- Configure appropriate deployment modes for different scenarios

---

*This guide provides the essential information for migrating from PSAppDeployToolkit v3 to v4. For detailed documentation, refer to the official PSAppDeployToolkit documentation.*