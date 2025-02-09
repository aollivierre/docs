# SQL Server Remote Connection Configuration Guide

## Environment

- SQL Server: CCI-SQL01
- Management Station: CCI-MGMT01
- Domain: cci.local

## Issue Description

Configuring remote SQL Server connections through Windows Firewall while maintaining security and resolving certificate trust issues.

## Steps Taken

### 1. Configure Windows Firewall

Created an inbound rule to allow SQL Server connections specifically from the management station:

```powershell
New-NetFirewallRule -DisplayName "Allow SQL Server from CCI-MGMT01" -Direction Inbound -Protocol TCP -LocalPort 1433 -Action Allow -RemoteAddress (Resolve-DnsName -Name CCI-MGMT01.cci.local).IPAddress
```

This command:

- Creates an inbound firewall rule
- Allows TCP port 1433 (SQL Server default port)
- Restricts access to only CCI-MGMT01
- Uses DNS resolution to get the management station's IP address

### 2. Certificate Trust Issue

After configuring the firewall, encountered certificate trust errors when connecting through SQL Server Management Studio (SSMS).

Error message received:

```text
A connection was successfully established with the server, but then an error occurred during the login process. 
(provider: SSL Provider, error: 0 - The certificate chain was issued by an authority that is not trusted.)
```

### 3. Solution

The issue was resolved by configuring SSMS to trust the server certificate:

1. In SQL Server Management Studio:

   - Click "Connect" → "Database Engine"
   - Enter server name: `CCI-SQL01`
   - Click "Options >>"
   - Go to "Connection Properties" tab
   - Check "Trust server certificate"
   - Click "Connect"

## Best Practices and Security Considerations

1. Firewall rule is specifically scoped to:

   - Only the required port (1433)
   - Only the management station IP
   - Only inbound traffic

2. Encryption remains enabled on SQL Server

   - Maintains secure, encrypted communications
   - Certificate trust configured on client side
   - No compromise to server-side security settings

## Additional Notes

For production environments, consider:

- Implementing a proper certificate authority (CA)
- Using domain-issued certificates
- Regular certificate maintenance and renewal
- Monitoring and auditing connection attempts
