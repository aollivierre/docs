# PowerShell Credential Management Guidelines

## Overview

This document provides guidance on credential management approaches in PowerShell projects, with a focus on when to use PowerShell's SecretManagement module versus custom solutions.

## General Rule

**Prefer using PowerShell SecretManagement for new projects unless there's a specific reason to use a custom solution.**

## Decision Flowchart

1. Does the project already have a custom credential management solution?
   - **Yes**: Continue using it unless there's a compelling reason to migrate
   - **No**: Use PowerShell SecretManagement

2. Does the project need to support PowerShell 5.1 or earlier?
   - **Yes**: Consider a custom solution or ensure SecretManagement compatibility
   - **No**: Prefer SecretManagement

3. Is minimizing dependencies critical?
   - **Yes**: Consider a custom solution
   - **No**: Use SecretManagement

## When to Use SecretManagement

- For new projects without existing credential management
- When centralized credential management is desired
- When credentials need to be shared across multiple modules
- When enterprise integration (Azure KeyVault, etc.) is needed
- When storing multiple types of secrets (not just credentials)
- When portability between machines is required (via KeePass extension)

## When to Use Custom Solutions

- When maintaining existing code with a working credential system
- When zero external dependencies are required
- When targeting environments where installing modules is restricted
- When only simple credential storage is needed
- When you need maximum control over the storage mechanism

## SecretManagement Implementation

```powershell
# Install required modules (one-time setup)
Install-Module Microsoft.PowerShell.SecretManagement -Scope CurrentUser
Install-Module Microsoft.PowerShell.SecretStore -Scope CurrentUser

# Register a vault (one-time setup)
Register-SecretVault -Name 'LocalStore' -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault

# Store a credential
$cred = Get-Credential
Set-Secret -Name 'ServiceCredential' -Secret $cred

# Retrieve a credential
$storedCred = Get-Secret -Name 'ServiceCredential'
```

## Custom Implementation Example

```powershell
# Store a credential
function Save-Credential {
    param (
        [Parameter(Mandatory = $true)]
        [System.Management.Automation.PSCredential]$Credential,
        
        [Parameter(Mandatory = $true)]
        [string]$Name
    )
    
    $credPath = Join-Path -Path $env:USERPROFILE -ChildPath ".credentials"
    if (-not (Test-Path -Path $credPath)) {
        New-Item -Path $credPath -ItemType Directory -Force | Out-Null
    }
    
    $credFile = Join-Path -Path $credPath -ChildPath "$Name.xml"
    $Credential | Export-Clixml -Path $credFile
}

# Retrieve a credential
function Get-StoredCredential {
    param (
        [Parameter(Mandatory = $true)]
        [string]$Name
    )
    
    $credPath = Join-Path -Path $env:USERPROFILE -ChildPath ".credentials"
    $credFile = Join-Path -Path $credPath -ChildPath "$Name.xml"
    
    if (Test-Path -Path $credFile) {
        Import-Clixml -Path $credFile
    } else {
        Write-Error "No credential found for $Name"
    }
}
```

## Key Considerations

1. **Security**: Both approaches use Windows DPAPI for local encryption
2. **Portability**: Both default stores are tied to the current user profile
3. **Extensibility**: SecretManagement offers more vault options
4. **Dependencies**: Custom solutions have fewer dependencies
5. **Standardization**: SecretManagement provides a consistent API

## Best Practices

1. Never hardcode credentials in scripts
2. Always use secure string objects for passwords
3. Limit credential access to only what's needed
4. Consider using certificate-based authentication where possible
5. For web services, prefer OAuth or token-based authentication
6. Document the credential management approach used in your project
