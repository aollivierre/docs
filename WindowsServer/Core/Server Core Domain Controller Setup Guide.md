# Server Core Domain Controller Setup Guide

This guide details the step-by-step process to configure a Windows Server 2022 Core as an additional Domain Controller.

## Prerequisites

- Windows Server 2022 Core installation completed with default admin password set
- Existing domain controller (DC1) running and accessible
- Network connectivity between servers
- Domain admin credentials

## Step 1: Initial Server Core Preparation

After fresh installation, you'll see the SConfig menu. Remote Management is enabled by default on Server Core installations, but you can verify this in SConfig (Option 4).

To get the current IP address:

1. Select Option 15 in SConfig to exit to PowerShell
2. Run the following command:

```powershell
Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.InterfaceAlias -notlike "*Loopback*" } | Format-Table InterfaceAlias, IPAddress, PrefixLength
```

## Step 2: Management Server Setup

On your management server:

1. Install RSAT tools by running:

```powershell
# For Windows 10/11:
Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability -Online

# For Windows Server:
Install-WindowsFeature RSAT-AD-Tools, RSAT-DNS-Server
```

2. Configure WinRM if not already set:

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force
```

## Step 3: Network Configuration

Run the server preparation script ([Server Core Preparation Script](https://github.com/aollivierre/HyperV/blob/41bcfa568d00712ba6e05659a32fc81b04031eb8/2-Create-HyperV_VM/Latest/Prepare%20Server%20Core%20Domain%20Controller/2-Automated%20Server%20Core%20Preparation%20with%20Current%20Network%20Settings.ps1)) which will:

1. Store admin credentials securely
2. Configure static IP settings
3. Set DNS to point to DC1
4. Rename computer to DC2
5. Join the domain

### Key Parameters Required

- DC1's IP address (for DNS)
- Desired static IP for DC2
- Domain admin credentials
- Domain name (e.g., ABC.local)

## Step 4: Disable Windows Firewall (if required in your environment)

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

Verify with:

```powershell
Get-NetFirewallProfile | Format-Table Name,Enabled
```

## Step 5: Promote to Domain Controller

After the server has restarted from domain join:

1. Run the [DC promotion script](https://github.com/aollivierre/HyperV/blob/41bcfa568d00712ba6e05659a32fc81b04031eb8/2-Create-HyperV_VM/Latest/Prepare%20Server%20Core%20Domain%20Controller/3-Add%20Additional%20Domain%20Controller%20to%20Existing%20Domain.ps1)
2. Provide SafeMode Administrator Password
3. Wait for promotion to complete and server to restart

## Step 6: Post-Promotion Tasks

Run the [diagnostics script](https://github.com/aollivierre/HyperV/blob/41bcfa568d00712ba6e05659a32fc81b04031eb8/2-Create-HyperV_VM/Latest/Prepare%20Server%20Core%20Domain%20Controller/4-Remote%20Post%20DC%20Promotion%20Diagnostics.ps1) to verify:

- DCDiag health checks
- Replication status
- DNS configuration
- FSMO roles
- Forest/Domain information
- Global Catalog status
- Critical services status

## Scripts Used in This Process

### 1. Server Core Preparation Script

```powershell
# Prepares server core with network settings and domain join
# Download from: https://github.com/aollivierre/HyperV/blob/41bcfa568d00712ba6e05659a32fc81b04031eb8/2-Create-HyperV_VM/Latest/Prepare%20Server%20Core%20Domain%20Controller/2-Automated%20Server%20Core%20Preparation%20with%20Current%20Network%20Settings.ps1
```

### 2. DC Promotion Script

```powershell
# Promotes server to domain controller
# Download from: https://github.com/aollivierre/HyperV/blob/41bcfa568d00712ba6e05659a32fc81b04031eb8/2-Create-HyperV_VM/Latest/Prepare%20Server%20Core%20Domain%20Controller/3-Add%20Additional%20Domain%20Controller%20to%20Existing%20Domain.ps1
```

### 3. Post-Promotion Diagnostics Script

```powershell
# Verifies DC health and configuration
# Download from: https://github.com/aollivierre/HyperV/blob/41bcfa568d00712ba6e05659a32fc81b04031eb8/2-Create-HyperV_VM/Latest/Prepare%20Server%20Core%20Domain%20Controller/4-Remote%20Post%20DC%20Promotion%20Diagnostics.ps1
```

## Important Notes

- Verify DNS settings are correct before attempting domain join
- Keep note of all passwords used during setup
- Maintain proper documentation of IP addresses and server roles

## Troubleshooting

### Common Issues

1. Domain join fails
   - Verify DNS settings point to DC1
   - Check network connectivity
   - Verify credentials

2. DC Promotion fails
   - Check network connectivity
   - Verify SYSVOL share access
   - Review event logs

3. Replication issues
   - Check firewall settings
   - Verify DNS configuration
   - Review replication event logs

### Verification Commands

```powershell
# Check DC health
dcdiag /s:DC2

# Check replication
repadmin /showrepl DC2

# Verify DNS
Get-DnsServerZone -ComputerName DC2
```

## Best Practices

1. Always run initial configuration from a secure management server
2. Store scripts in a version-controlled repository
3. Document all IP addresses and configurations
4. Maintain secure password storage
5. Regular backup of system state
6. Monitor replication health

## Security Considerations

1. Enable firewall after initial setup (if disabled)
2. Use strong passwords
3. Regular security updates
4. Monitor security logs
5. Follow least privilege principle
