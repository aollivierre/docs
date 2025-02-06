# Windows Server DNS and SQL Firewall Configuration Guide

This guide documents the process of configuring DNS records and SQL Server firewall rules for communication between Windows SQL Server and Ubuntu Server in an Active Directory environment.

## System Information

- Domain: cci.local
- Domain Controller: CCI-DC01 (10.224.10.10)
- SQL Server: CCI-SQL01 (10.224.10.11)
- Ubuntu Server: CCI-LAYERNEXT01 (10.224.10.9)

## DNS Configuration

### Step 1: Create Forward DNS Record

Create an A record for the Ubuntu server in the domain:

```powershell
Add-DnsServerResourceRecordA -Name "CCI-LAYERNEXT01" -ZoneName "cci.local" -IPv4Address "10.224.10.9" -CreatePtr
```

If this fails with PTR record creation error, proceed with steps 2 and 3.

### Step 2: Create Reverse Lookup Zone

Create the reverse lookup zone for your subnet:

```powershell
Add-DnsServerPrimaryZone -NetworkID "10.224.10.0/24" -ReplicationScope "Domain"
```

### Step 3: Create PTR Record

Add the PTR record manually:

```powershell
Add-DnsServerResourceRecordPtr -Name "9" -ZoneName "10.224.10.in-addr.arpa" -PtrDomainName "CCI-LAYERNEXT01.cci.local"
```

### Step 4: Verify DNS Resolution

Test both forward and reverse DNS resolution:

```powershell
# Test forward lookup
Resolve-DnsName -Name CCI-LAYERNEXT01.cci.local

# Test reverse lookup
Resolve-DnsName -Name 10.224.10.9
```

## SQL Server Firewall Configuration

### Step 1: Add Firewall Rule

After DNS is properly configured, create the firewall rule to allow SQL Server access:

```powershell
New-NetFirewallRule -DisplayName "Allow SQL Server from CCI-LAYERNEXT01" -Direction Inbound -Protocol TCP -LocalPort 1433 -Action Allow -RemoteAddress (Resolve-DnsName -Name CCI-LAYERNEXT01.cci.local).IPAddress
```

Alternative method using direct IP (if DNS is not available):

```powershell
New-NetFirewallRule -DisplayName "Allow SQL Server from CCI-LAYERNEXT01" -Direction Inbound -Protocol TCP -LocalPort 1433 -Action Allow -RemoteAddress 10.224.10.9
```

### Step 2: Verify Firewall Rule

Check the created firewall rule:

```powershell
Get-NetFirewallRule -DisplayName "Allow SQL Server from CCI-LAYERNEXT01" | Format-List
```

## Verification

### Test Connectivity

From the Ubuntu server, test SQL Server connectivity:

```bash
# Using telnet
telnet cci-sql01.cci.local 1433

# Or using nc (netcat)
nc -zv cci-sql01.cci.local 1433
```

## Troubleshooting

### Common Issues

1. **DNS Resolution Failures**
   - Verify DNS server settings on both servers
   - Check for correct A and PTR records
   - Ensure reverse lookup zone exists

2. **Firewall Issues**
   - Verify firewall rule is enabled
   - Check if the correct ports are open
   - Confirm IP addresses are correct

3. **SQL Server Access**
   - Ensure SQL Server is listening on TCP/1433
   - Check SQL Server authentication settings
   - Verify network protocols are enabled

### Useful Commands

**DNS Troubleshooting:**
```powershell
# List all DNS zones
Get-DnsServerZone

# View specific DNS records
Get-DnsServerResourceRecord -ZoneName "cci.local" -Name "CCI-LAYERNEXT01"

# Check reverse lookup zone
Get-DnsServerResourceRecord -ZoneName "10.224.10.in-addr.arpa"
```

**Firewall Troubleshooting:**
```powershell
# List all SQL Server related rules
Get-NetFirewallRule | Where-Object { $_.DisplayName -like "*SQL*" } | Format-Table DisplayName, Enabled, Direction, Action

# Test network connectivity
Test-NetConnection -ComputerName CCI-LAYERNEXT01.cci.local -Port 1433
```

## Security Considerations

1. **Firewall Rules**
   - Only open necessary ports
   - Use specific IP addresses instead of ranges when possible
   - Regularly audit firewall rules

2. **DNS Security**
   - Keep DNS records up to date
   - Remove stale DNS entries
   - Monitor DNS changes

3. **SQL Server Security**
   - Use SQL Server authentication best practices
   - Implement least privilege access
   - Regular security audits

## Maintenance

Regular maintenance tasks:
1. Verify DNS records are current
2. Check firewall rule effectiveness
3. Monitor SQL Server connectivity
4. Update documentation when changes occur

## Related Documentation

- [Ubuntu Server Active Directory Integration Guide](./Ubuntu%20Server%20Active%20Directory%20Integration%20Guide.md)
- [Microsoft SQL Server Documentation](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access)
- [Windows Server DNS Documentation](https://docs.microsoft.com/en-us/windows-server/networking/dns/dns-top) 